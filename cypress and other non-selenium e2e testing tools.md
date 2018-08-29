# There is life after Selenium
- [There is life after Selenium](#there-is-life-after-selenium)
  - [Intro](#intro)
  - [Cypress](#cypress)
    - [Concepts](#concepts)
    - [Architecture](#architecture)
    - [Installation](#installation)
    - [Usage](#usage)
      - [Cypress configuration](#cypress-configuration)
      - [Writing tests in Cypress](#writing-tests-in-cypress)
      - [Test framework design with Cypress](#test-framework-design-with-cypress)
      - [Cypress in CI](#cypress-in-ci)
    - [Cypress Dashboard Service](#cypress-dashboard-service)
    - [Other bells and rings](#other-bells-and-rings)
    - [What can be done better](#what-can-be-done-better)
  - [Puppeteer](#puppeteer)
    - [Concepts](#concepts)
    - [Writing tests](#writing-tests)
    - [Installation](#installation)
    - [Other bells and rings](#other-bells-and-rings)
    - [Drawbacks](#drawbacks)
  - [TestCafe](#testcafe)
    - [Concepts](#concepts)
    - [Installation](#installation)
    - [Writing tests](#writing-tests)
    - [Other bells and rings](#other-bells-and-rings)
  - [Summary](#summary)
    - [Tools comparison](#tools-comparison)
- [Links](#links)


## Intro
While there is nothing horribly wrong with Selenium, it's a great tool and many people successfully use it, I believe there are other tools we should be aware of. So we can make an informed decision when building a test automation solution on your new shiny project.

In this article, I will highlight 3 major open-source non-selenium test automation frameworks. I will try to compare them by categories: use, x-browser support, community support etc.

## [Cypress](https://www.cypress.io/)

I worked with cypress for a bit over a year, since beta 0.2x. It's grown in quite a mature framework, that can fulfill many of the test automation needs.

### Concepts

The main idea behind Cypress is that it bundles a bunch of existing testing frameworks in one solution. So you have all of your mocks, stubs, assertions as one framework.

Another thing that makes cypress stand-out is that it operates within your application: its node.js server runs in the same run-loop of your browser and not in remote session with some network wiring (like JSON wire protocol of Selenium or DevTools protocol of Puppeteer). So it has full control over your application, over traffic to it and can execute any off-browser actions, as it's node.js application.

Cypress does not target to replace all of your unit-tests or integration tests frameworks, you still can do it in cypress as helpers. But the main focus of cypress is e2e.

Cypress tests are written in JS, you can also use typescript.

The main reason why I choose cypress - it's incredible dev-friendly. I think those old days when you have devs throwing code/applications to QA and they manage quality and automation already long gone. The only way you can scale any test automation - is to involve developers from day 1. And for that, the framework should be with dev-first mentality.

### Architecture

Cypress main parts are:
- Runner - chrome app, glue between the driver, reporter, server, and extension 
- Driver - the library that loads inside of your browser and executes commands in runtime
- Extension - chrome extension
- Server - node app behind the browser, responsible for all communication and external processes (reporters, plugins, other node processes)
- Reporter - UI component for showing process and test results

### Installation

`npm install cypress`. That's it. It supports win/lin/mac as hosts. If you want, you can also download a stand-alone application if for some reason you don't want to mess with node/npm for just running tests.

### Usage

#### Cypress configuration
Cypress setup is defined in `cypress.json`. There you can set up base URL of your page, timeouts, including and excluding policies, video recordings and a bunch of other settings. Same you can set up in .env.json files and as ENV variables or run parameters for cypress command line.

`support/defaults.js` - for overwriting defaults of cypress. F.e. I use it for whitelisting some cookies that I persist between runs

`support/index.js` - global config and behaviour of cypress. F.e. there I overwrite `before()` method, as I need to have it by default before all tests

`support/assertions.js`  - for extending assertions that you can use in your tests. F.e. I extend it with `chai-almost` assertion.

`support/commands.js` - for extending cypress commands. F.e. you can extend cypress with your custom `cy.login()` command for some common actions you execute. I don't use it as I prefer to have my own commands and methods rather than extending cypress. 

`support/index.js` - for your custom plugins. It's executed when you (re)open project. I use it to load cypress configs for different environments.

#### Writing tests in Cypress 

Cypress tests are mocha.js tests. All of BDD syntax of mocha (I'm talking about grouping tests in sets by `describe()`, and tests are located in `it()` methods, `before()` and `after()` hooks etc..) without overhead of Cucumber BDDs, although there is a plugin for it https://github.com/TheBrainFamily/cypress-cucumber-preprocessor

Tests are located in the `integration` folder. The simplest test will look like:
```javascript
describe('My First Test', function() {
  it('Does not do much!', function() {
    expect(true).to.equal(true)
  })
})
```

Cypress bundles jQuery, so your locators will look like
```javascript
cy.get('#main-content')
  .find('.article')
  .children('img[src^="/static"]')
  .first()
```

One of the core concepts of cypress - tests are deterministic. You should not have conditional testing, you should not have any recovery scenarios if you can't find the element, like
```javascript
if ($myElement.length) {
  doSomething($myElement)
}
```
If the element is not found - the test will fail. Period. This is the only way to force you tests be not flaky, yourself to be consistent and your application - stable.

The side benefit of this behaviour resolves in fact that you need fewer assertions.

Assertions are chai assertions, and looks like ```cy.get('button').click().should('have.class', 'active')```

#### Test framework design with Cypress

In cypress docs, you can find a lot of mentions about focusing on simplicity. I do agree with that and at same, I believe in code deduplication and DRY.

There is much of holly believes for PageObjects in QA community. Cypress will not block or support you if you want to implement it. I did it for my project as it was convenient at the beginning, we also build one or 2 levels of abstractions on top of PO, we call it `flows`.

More cypress-native way to decouple repeating logic is to put it in `support/commands.js`, so you are extending `cy()` object with custom actions, like login/logouts, create users etc.

Another point is to bypass UI as much as you can, using `cy.request()` calls to seed your test session with anything you need (user tokens/cookies) and stubbing.


#### Cypress in CI

Cypress integrates into CI quite easily, there are a bunch of [examples for most of CI providers](https://docs.cypress.io/guides/guides/continuous-integration.html#What-is-supported). 
Best to run cypress as docker container, there are pre-built images or you can build it yourself from [examples](https://github.com/cypress-io/cypress-docker-images)

If you don't use Dashboard service, you can publish cypress artifacts and use any `mocha-reporter`, https://github.com/adamgruber/mochawesome is my favorite so far.

Watch out for cache folder, which cypress use for installation -  if you have a multi-job pipeline, you have to pass it as cashed folder or artifact to upstream jobs.

With version 3.1.0 you can set up parallel runs with your CI runs using Cypress Dashboard Service

### Cypress Dashboard Service
Dashboard service from cypress allows you to look results of test runs. It has some history, video recordings, and screenshots + test failure reasons in case of any. 

With new cypress 3.1.0, Dashboard service also behaves as scheduler/dispatcher for parallel runs of your tests. Before all tests runs were sequential, which is not best for debugging, but now it's much improved.

### Other bells and rings

Cypress have quite cool **time travel** feature. It allows you to look back on the state of application and DOM, which can be quite helpful for debugging. 

In general, debugging is one of the strong sides of cypress, you can use regular `debugger()` inside your tests to break in test execution and add a hook to Chrome debugger.

There is UI inspector that can help you to build actions and locators (if you need this kind of support)

[Stubs, spies and clocks](https://docs.cypress.io/guides/guides/stubs-spies-and-clocks.html#Spies) can help you to inject some state (if you work with Redux or some other state machine), simulate network responses and error handling (how your app behaves on 500 error) and fully control time in your application (to fast-forward animation)

Cypress waits for the element to be on the page automatically, so you don't need to make any async functions to support relative waits by yourself.

### What can be done better
What I really miss is an out-of-the-box rerunner for failed tests. Cypress is way less flaky as selenium,f.e. network lag is removed as cypress lives in the same runtime of the browser.
You can implement re-run using cypress Module API, but it's not as easy and native as I would like to see it, and you need to make a choice if you want parallel run or re-run.

Because of fundamental technical choices cypress team did, it's fairly impossible to run tests on non-chromium browsers. 

There are a bunch of other trade-offs you should be aware https://docs.cypress.io/guides/references/trade-offs.html#Temporary-trade-offs

## Puppeteer

### Concepts

This is node.js library that allows you to control your Chromium-based browser over DevTools protocol. Puppeteer is not fully bundled framework as Cypress, which give you both full control over your test automation framework choices, and at the same time creates a bit of overhead, as you have to add stuff yourself. 
It's can be more familiar for anyone why was using Selenium before, as you operate with `Browser` object instance for any actions. 

Test executes in the headless chromium browser, but you can configure to start it in windowed mode or start with chrome. Differences between browsers https://www.howtogeek.com/202825/what%E2%80%99s-the-difference-between-chromium-and-chrome/

I think it Puppeteer was developed for dogfooding and testing DevTools of chrome by chrome team :)

Recently I saw a lot of non-test related use of puppeteer, for example, it used a lot for site crawlers, f.e. https://github.com/yujiosaka/headless-chrome-crawler  or system health monitors like https://checklyhq.com/. In general, I see puppeteer as an amazing browser-automation tool, not a tool I would use for test automation.

### Writing tests

Tests are simple js files, so by default, it's just code without mocha stile test grouping or assertions.

There are some plugins already exists, like https://github.com/smooth-code/jest-puppeteer or https://github.com/direct-adv-interfaces/mocha-headless-chrome

Comparing to cypress which makes some magic with promises and making async code looking synchronous, in puppeteer you have to really use await-async everywhere



### Installation

As any node.js library - `yarn add puppeteer`

### Other bells and rings

You can test your Chrome extensions. Same stuff you can do with cypress with a bit of a hack https://github.com/cypress-io/cypress/issues/1965

With timeline trace, you can test the performance of the application.

There is a slowMo config option that allows you to lower down execution speed of tests for better debugging.

You can hook up on console events of the browser, so you can better log

In page evaluation, you can set debugger so your browser instance will start with DevTools and debugger attached.

Puppeteer sandbox can help you to get started fast https://puppeteersandbox.com/

### Drawbacks 

Puppeteer is bundled with specific chrome version, so if you need to test on new chrome, you have to use new puppeteer version as well.



## TestCafe

### Concepts
TestCafe as the framework has a lot of similarities with cypress. Their both run in browser runtime, 
both of them support JS or TS as test language. TestCafe tries to be closer to QA, even to manual QA, as it has TestCafe Studio - the tool for record&playback tests with scriptless context. Another thing which will be very familiar to anyone in QA field is the PageObject pattern, as it builds in the TestCafe framework.
TestCafe is only one truly x-browser testing tool from this comparison. 

TestCafe support parallel test execution out of the box.

### Installation

`yarn add testcafe`, no surprises here.

`testcafe chrome tests/` - to run the test in `test` folder.


### Writing tests

You can write in TS or JS. Looks like TestCafe has its own syntax for tests and assertions, so it's not as quick to adopt for devs as in case of Cypress which is bundled with mocha and chai. Tests sets called `features` that starts from a specific `page`, inside of features you define your tests.

Simple test will look like:
```javascript
import { Selector } from 'testcafe'; // first import testcafe selectors

fixture `Getting Started`// declare the fixture
    .page `https://devexpress.github.io/testcafe/example`;  // specify the start page


//then create a test and place your code there
test('My first test', async t => {
    await t
        .typeText('#developer-name', 'John Smith')
        .click('#submit-button')

        // Use the assertion to check if the actual header text is equal to the expected one
        .expect(Selector('#article-header').innerText).eql('Thank you, John Smith!');
});
```
### Other bells and rings

With testcafe-live, you can write tests and instantly see a result as you go, similar to filesystem watch in Cypress.


## Summary

Cypress is a tool built by developers for **developers** for **testing**

TestCafe is a tool build by developers/QA for **QA**.

Puppeteer is a tool built by developers for developers for **browser automation**.

If test automation for you is a side effect, but browser-automation is a must, use Puppeteer. If you don't want to spend time on fine-tuning, and mainly want to test your app - use Cypress or TestCafe. If x-browser support is a must - use TestCaffe. If you want to build dev-centric test automation framework - use Cypress.

### Tools comparison

| Criteria          | Cypress         | TestCaffe                                                                                                                              | Puppitier       |
| ----------------- | :-------------: | -------------------------------------------------------------------------------------------------------------------------------------: | --------------: |
| Language support  | JS              | JS/Typescipt                                                                                                                           | JS              |
| Browser Support   | Chrome/chromium | [Many main browsers](https://devexpress.github.io/testcafe/documentation/using-testcafe/common-concepts/browsers/browser-support.html) | Chrome/chromium |
| Pricing           | Free(beta)      | [499$](https://testcafe.devexpress.com/Buy/)                                                                                           | Free            |
| Extendability     | Moderate        | Low, many things is propreitary                                                                                                        | High            |
| Examples (simple) |                 |                                                                                                                                        |                 |


# Links
- https://docs.cypress.io/api/cypress-api/custom-commands.html#Best-Practices
- https://docs.cypress.io/faq/questions/using-cypress-faq.html
- https://docs.cypress.io/guides/core-concepts/writing-and-organizing-tests.html
- https://github.com/cypress-io/testing-workshop-cypress - nice workshop to follow full integration of cypress 
- https://github.com/cypress-io/cypress-example-recipes more than enough of recipes to get you started

