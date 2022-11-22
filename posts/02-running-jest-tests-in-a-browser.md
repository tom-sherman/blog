---
title: "Running Jest Tests in a Real Browser"
createdAt: "2020-07-25"
tags: ["testing", "jest", "javascript"]
---

# Running Jest Tests in a Real Browser

If you didn't know, Jest is in my opinion the best testing framework for modern JavaScript and TypeScript. The developer experience is so smooth and lightning fast (lightning smooth?) which makes chasing that holy grail of 100% unit test coverage a breeze. Unfortunately, like many things, unit tests do not offer you 100% protection. To explain why I'd love for you to indulge me while I tell a very short story;

## The unit test footgun

So picture this; I'm writing a new feature, following the rules, obeying TDD, watching the terminal light up green as I add code for my test cases. I finish writing my feature and all of my tests pass, great! It's [not Friday](https://twitter.com/kvlly/status/1098941288727093248?lang=en) so I deploy my changes to production, all of the logs look great so I go to bed after another successful day of being a [10x developer](https://medium.com/@stevenpcurtis.sc/the-dangerous-myth-of-the-10x-developer-534d0e9159c9#:~:text=Terminology,job%20specification%20and%20limited%20benefit).

The next day I log on to see a critical bug has been raised relating to the perfect feature I finished yesterday. How could this happen? Weren't all of my tests green? Some of you may know where this is going. After diagnosing the bug for a couple of hours I find that the code I've written uses some relatively new feature of some Web API that isn't supported in several of my user's browsers. I slap myself on the back of the wrist for not considering this before deploying and get to work on a fix. But I ask myself, why wasn't this caught by the unit tests?

The answer is down to Jest not being a _real_ browser. You have access to all of the various DOM APIs when writing Jest tests but these are provided by [JSDOM](https://github.com/jsdom/jsdom) and your tests are actually running in Node. Jest has no native way to run your tests in a browser environment out of the box.

## Some alternative solutions before we get to the good stuff

There are some ways to solve this. You could have a separate test suite using a different framework that can run in a real browser. In my experience though, this is difficult to sell to other developers. Now we have to write a load of tests twice, one for Jest to get tight feedback loops and once for the browser. What makes this worse is that usually your browser testing framework will have a completely different API and sometimes need to be written in a completely different language.

Another solution could be to use [jest-puppeteer](https://github.com/smooth-code/jest-puppeteer). This is a great tool for spawning a headless version of Chrome and using the puppeteer API to write your tests inside of Jest. This technique works well as you're still in the Jest ecosystem, but now you have to choose if a test needs to be written using puppeteer or if it's ok to run in a Node and JSDOM environment. It also doesn't help if you've already got hundreds (or thousands) of tests written for your application.

So in summary, we're looking for a solution that:

- Runs in real browsers that reflect the actual users of the application
- Doesn't waste our time by making us write tests twice or choose between environments
- Allow us to take advantage of the Jest APIs

## The good stuff

Here's the TL;DR of how we have our cake and eat it too:

Write all of your tests once using Jest as your framework. These will run in Node and JSDOM for local development to keep feedback loops tight. Then, usually part of CI, run these exact same tests in a browser environments.

### Step 1: Install Karma and related packages

[Karma](https://karma-runner.github.io/latest/index.html) is an amazingly versatile test runner - here we'll be using it to compile our tests, spawn a real life browser, and report our successes and failures. We'll also need some plugins to get everything working too.

```
npm i karma karma-jasmine webpack karma-webpack expect jest-mock -D
```

We'll be using [karma-jasmine](https://github.com/karma-runner/karma-jasmine) because it's top-level API is almost identical to Jest's. We'll also be using [karma-webpack](https://github.com/webpack-contrib/karma-webpack) to bundle our tests together so they can be used in the browser.

### Step 2: Create a `karma.config.js`

```javascript
module.exports = function (config) {
  config.set({
    plugins: ["karma-webpack", "karma-jasmine"],

    // base path that will be used to resolve all patterns (eg. files, exclude)
    basePath: "",

    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ["jasmine"],

    // list of files / patterns to load in the browser
    // Here I'm including all of the the Jest tests which are all under the __tests__ directory.
    // You may need to tweak this patter to find your test files/
    files: ["__tests__/**/*.js"],

    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      // Use webpack to bundle our tests files
      "packages/*/__tests__/**/*.ts": ["webpack"],
    },
  });
};
```

### Step 3: Add the webpack config

You can use one that you already have for your application or configure one that is based off of your Jest babel config.

```javascript
module.exports = function (config) {
  config.set({
    // ...
    webpack: {
      // Your webpack config here
    },
    // ...
  });
};
```

Creating a Webpack config is well and truly outside the scope of this article because everyone has different setups. You can read more in the [Webpack](https://webpack.js.org/concepts/) and [karma-webpack docs](https://github.com/webpack-contrib/karma-webpack).

### Step 4: Add a `karma-setup.js`

This is where the magic happens. There are things that Jest provides as part of the global API that is not available in Jasmine. Examples are the `expect` [matchers API](https://jestjs.io/docs/en/using-matchers) and the `jest.fn()` [mock/stubbing API](https://jestjs.io/docs/en/mock-functions#using-a-mock-function). So we are going to include a file in our test bundle that adds these APIs to the global scope.

```javascript
// the jest.fn() API
import jest from "jest-mock";
// The matchers API
import expect from "expect";

// Add missing Jest functions
window.test = window.it;
window.test.each = (inputs) => (testName, test) =>
  inputs.forEach((args) => window.it(testName, () => test(...args)));
window.test.todo = function () {
  return undefined;
};
window.jest = jest;
window.expect = expect;
```

Note that I have defined the parts of the Jest API I need, so if you use other parts then you may need to implement or import those too. The only thing that is not possible to use is the [module mock API](https://jestjs.io/docs/en/jest-object#mock-modules).

#### Add the `karma-setup.js` file to the "preprocessors" and "files" config arrays.

```javascript
// karma.conf.js

module.exports = function (config) {
  config.set({
    // ...
    files: ["./scripts/karma-setup.js", "packages/*/__tests__/**/*.ts"],

    preprocessors: {
      "./karma-setup.js": ["webpack"],
      "packages/*/__tests__/**/*.ts": ["webpack"],
    },
    // ...
  });
};
```

### Step 5: Install browsers and browser launchers

You will of course need the browsers installed on your system in order to run the tests on them. After installing them, you'll need to install the associated browser launcher.

You can find one on npm here: https://www.npmjs.com/search?q=keywords:karma-launcher

I'm going to setup Chrome for the tests so I'll install [karma-chrome-launcher](https://github.com/karma-runner/karma-chrome-launcher):

```
npm i karma-chrome-launcher -D
```

#### And then add it to the karma.conf.js configuration:

```javascript
// karma.conf.js

module.exports = function (config) {
  config.set({
    // ...
    plugins: [
      "karma-webpack",
      "karma-jasmine",
      // Adding it to the plugins array
      "karma-chrome-launcher",
    ],

    // I'm starting a headless browser, but I can also swap this out for "Chrome" to add debug statements, inspect console logs etc.
    browsers: ["ChromeHeadless"],
    // ...
  });
};
```

### Step 6: Run the tests!

Add a script to your package.json scripts, maybe call it "browser-tests"

```json
{
  "scripts": {
    "browser-tests": "karma start"
  }
}
```

And then run `npm run browser-tests` to the start the tests.

## Summary

Behold your beautiful browser tests in their full glory! Now you have one Jest test suite running in Node for a great DX and in the browser for true integration tests.

Now, this is a lot of setup but you do get massive benefits at the end of it. I devised this technique when we had many Jest tests and we wanted to run them in a real browser without rewriting them all. I think we ended up with a real solid solution.

I'd love to know if this solution worked (or didn't work) for you, and if you have any questions just let me know!

---

Discuss this post in the [GitHub discussion](https://github.com/tom-sherman/blog/discussions/6) or [on Twitter](https://twitter.com/tomus_sherman/status/1287153112172638209).
