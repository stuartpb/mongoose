# Testing Mongoose with [Jest](https://www.npmjs.com/package/jest)

Jest is a client-side JavaScript testing library developed by Facebook.
Because Jest is designed primarily for testing React applications, using it to test Node.js server-side applications comes with a lot of caveats.
We strongly advise against using Jest for testing any Node.js apps unless you are an expert developer with an intimate knowledge of Jest.

If you choose to delve into dangerous waters and test Mongoose apps with Jest, here's what you need to know:

<h2 id="recommended-testenvironment"><a href="#recommended-testenvironment">Recommended <code>testEnvironment</code></a></h2>

Do **not** use Jest's default [`jsdom` test environment](https://jestjs.io/docs/en/configuration.html#testenvironment-string)
when testing Mongoose apps, _unless_ you are explicitly testing an
application that only uses
[Mongoose's browser library](https://mongoosejs.com/docs/browser.html).

The `jsdom` test environment attempts to create a browser-like test
environment in Node.js, and it comes with numerous nasty surprises like a
[stubbed `setTimeout()` function](https://github.com/jsdom/jsdom/commit/3f306bea5362aceb2a219a2e98ff96a7464d2f19#commitcomment-31316213)
that silently fails after tests are finished. Mongoose does not support jsdom
in general and is not expected to function correctly in the `jsdom` test
environment.

To change your `testEnvironment` to Node.js, add `testEnvironment` to your
`jest.config.js` file:

```javascript
module.exports = {
  testEnvironment: 'node'
};
```

<h2 id="timer-mocks"><a href="#timer-mocks">Timer Mocks</a></h2>

Absolutely do **not** use [timer mocks](https://jestjs.io/docs/en/timer-mocks.html)
when testing Mongoose apps. Fake timers stub out global functions like
`setTimeout()` and `setInterval()`, which causes problems when an underlying
library uses these functions. The MongoDB Node.js driver uses these functions
for deferring work until the next tick of the event loop and for monitoring
connections to the MongoDB server.
    
Mongoose devs have already refactored out code
to [avoid using `setImmediate()`](https://github.com/Automattic/mongoose/issues/6074)
to defer work to the next tick of the event loop, but we can't reasonably
ensure that every library Mongoose depends on doesn't use `setImmediate()`.

Mongoose uses `nextTick()`, which [Jest's underlying dependency explicitly doesn't stub by default](https://sinonjs.org/releases/latest/fake-timers/).
But, `process.nextTick()` isn't the same as `setImmediate()` because of
[microtasks vs macrotasks](https://abc.danch.me/microtasks-macrotasks-more-on-the-event-loop-881557d7af6f#2261),
so underlying libraries like the [MongoDB driver](http://npmjs.com/package/mongodb) may use `setImmediate()`.

To work around this, create your own wrapper around `setTimeout()` and
stub that instead using [sinon](http://npmjs.com/package/sinon).

```javascript
// time.js
exports.setTimeout = function() {
  return global.setTimeout.apply(global, arguments);
};

// Tests
const time = require('../util/time');
const sinon = require('sinon');
sinon.stub(time, 'setTimeout');
```

<h2 id="globalsetup-and-globalteardown"><a href="#globalsetup-and-globalteardown"><code>globalSetup</code> and <code>globalTeardown</code></a></h2>

Do **not** use `globalSetup` to call `mongoose.connect()` or
`mongoose.createConnection()`. Jest runs `globalSetup` in
a [separate environment](https://github.com/facebook/jest/issues/7184),
so you cannot use any connections you create in `globalSetup`
in your tests.

## Further Reading

Want to learn more about testing Mongoose apps? The
[RESTful Web Services with Node.js and Express](https://pluralsight.pxf.io/c/1321469/424552/7490?u=https%3A%2F%2Fapp.pluralsight.com%2Flibrary%2Fcourses%2Fnode-js-express-rest-web-services%2Ftable-of-contents)
course on Pluralsight has a great section on testing Mongoose apps with [Mocha](http://npmjs.com/package/mocha).

<a href="https://pluralsight.pxf.io/c/1321469/424552/7490?u=https%3A%2F%2Fapp.pluralsight.com%2Flibrary%2Fcourses%2Fnode-js-express-rest-web-services%2Ftable-of-contents">
  <img src="https://i.imgur.com/KouuaAZ.png">
</a>