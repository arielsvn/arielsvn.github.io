---
title: Set up end-to-end tests with `jest-pupeteer` and `NextJS`
date: 2020-02-29 09:00:01 Z
categories:
  - javascript
  - end-to-end tests
  - jest
  - jest-pupeteer
layout: post
summary: This contains details and the configuration files required to setup `jest-pupeteer` for end-to-end testing in a `NextJS` app.
---

Recently I started [migrating refinebio](https://github.com/AlexsLemonade/refinebio-frontend/pull/853) to [NextJS](https://nextjs.org/). It had originally been bootstrapped using [Create React App](https://create-react-app.dev/). There was one challenge that I couldn't find a lot of information online, and it was configuring end-to-end tests with [jest-pupetteer](https://github.com/smooth-code/jest-puppeteer). I'm going to write another blog post about other problems that I had to solve to complete the migration.

To run your tests with `jest-pupeteer` you need to configure 3 files. First, be sure to have a script to run a development server in `packages.json`

```js
"scripts": {
  "dev": "next",
  // [... other scripts]
},
```

Then, `jest-puppeteer.config.js` should have the following configuration.

```js
const PORT = 14568;

module.exports = {
  launch: {
    headless: process.env.CI === 'true',
  },
  browserContext: process.env.INCOGNITO ? 'incognito' : 'default',
  server: {
    command: `yarn run dev --port ${PORT}`,
    port: PORT,
    launchTimeout: 10000,
  },
};
```

This would ensure that a NextJS server runs in the port `14568` while the tests are running. It will run in `Headless` mode when the tests are executed in a continuous integration environment like circleci.

Finally, the jest configuration needs to be updated in `jest.config.js`.

```js
module.exports = {
  preset: 'jest-puppeteer',
};
```

### Conclusion

Even though there're several alternatives to running end-to-end tests for a NextJS application, the best feature of `jest-pupeteer` is that it's independent of `NextJS` itself. It can be set up to run any development server and the tests in parallel without much hassle.

