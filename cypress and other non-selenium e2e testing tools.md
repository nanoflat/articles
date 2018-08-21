# There is life after Selenium
- [There is life after Selenium](#there-is-life-after-selenium)
    - [Intro](#intro)
    - [Cypress](#cypress)
        - [Concepts](#concepts)
        - [Installation](#installation)
        - [Usage](#usage)
            - [Cypress coniguration](#cypress-coniguration)
            - [Writing tests in Cypress](#writing-tests-in-cypress)
            - [Test framework design with Cypress](#test-framework-design-with-cypress)
            - [Cypress in CI](#cypress-in-ci)
        - [Services](#services)
        - [Other bells and rings](#other-bells-and-rings)
        - [What can be done better](#what-can-be-done-better)
    - [Summary](#summary)
        - [Tools comparison](#tools-comparison)


## Intro
While there is nothing horribly wrong with Selenium, it's a great tool and many people sucessfully use it, I beleive there are other tools you we should be aware of. So we can make informed decision when building test automation solution on your new shiny project.

In this article I will highlight 3 major open-source non-selenium test automation frameworks. I will try to compare them by categories: use, x-browser support, community support etc.

## Cypress

I worked with cypress for a bit over a year, since beta 0.2x. It's grown in quite mature framework, that can fulfill many of test automation needs.

### Concepts

Main idea behind Cypress is that it bundles bunch of existing testing frameworks in one solution. So you have all of your mocks, stubs, assertions as one framework.

Another thing that makes cypress stand-out is that it runs in same run-loop of your browser and not in remote session with some network wiring. So it has full control over your application, over traffic to it and can execute any off-browser actions, as it's node.js application.

Cypress do not target to replace all of your unit-tests or integration tests frameworks, you still can do it in cypress as helpers. But main focus of cypress is e2e.

Cypress tests are written in JS, you can also use typescript.

Main reason why I choose cypress - it's incredible dev-friendly. I think that old days when you have devs throwing code/applications to QA and they manage quality and automation already long gone. The only way you can scale any test automation - is to involve developers from day 1. And for that, framework should be with dev-first mentality.

### Installation

`npm install cypress`. Thats it. It supports win/lin/macos as hosts. If you want, you can also download stand-alone application if for some reason you don't want to mess with node/nmp for just running tests.

### Usage

#### Cypress coniguration
Cypress setup is defined in cypress.json. There you can setup baseurl of your page, timeouts, including and excluding policies, video recordings and bunch of other settings. Same you can setup in .env.json files and as ENV variables or run parameters for cypress commandline.

`support/defaults.js` - for overwriting defaults of cypress. F.e. I use it for whitelisting some cookies that I persist between runs

`support/index.js` - global config and behaviour of cypress. F.e. there I overwrite `before()` method, as I need to have it by default before all tests

`support/assertions.js`  - for extending assertions that you can use in your tests. F.e. I extend it with `chai-almost` assertion.

`support/commands.js` - for extending cypress commands. F.e. you can extend cypress with your custom `cy.login()` command for some common actions you execute. I don't use it as I prefer to have my own commands and methods rather then extending cypress. 

`support/index.js` - for your custom plugins. It's executed when you (re)open project. I use it to load cypress configs for different environments.

#### Writing tests in Cypress 

Cypress tests are mocha.js tests. All of BDD syntax of mocha (I'm talking about grouping tests in sets by `describe()`, and tests are located in `it()` methods, `before()` and `after()` hooks etc..) without overhead of Cucumber BDDs, althow there is a plugin for it https://github.com/TheBrainFamily/cypress-cucumber-preprocessor

Test are located in `integration` folder. Simplest test will look like:
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

One of core concepts of cypress - tests are deterministics. You should not have conditional testing, you should not have any recovery scenarios if you can't find element, like
```javascript
if ($myElement.length) {
  doSomething($myElement)
}
```
If element is not found - fail test. Period. This is only way to force you tests be not flaky, yourself to be consistent and your application - stable.

Side benefit of this behaviour resolves in fact that you need less assertions.

Assertions are chai assertions, and looks like ```cy.get('button').click().should('have.class', 'active')```

#### Test framework design with Cypress

In cypress docs you can find a lot of mentions about focusing on simplicity. I do agree with that and at same I beleive in code deduplication and DRY.

There is much of holly beleives for PageObjects in QA community. Cypress will not block or support you if you want to implement it. I did it for my project as it was convinient at the beggining, we also build one of 2 levels of abstractions on top of PO, we call it `flows`.

More cypress-native way to decouple repeating logic is to put it in `support/commands.js`, so you are extending `cy()` object with custom actions, like login/logouts, create users etc.

Another point is to bypass UI as much as you can, using `cy.request()` calls to seed your test session with anything you need (user tokens/cookies) and stubbing.


#### Cypress in CI

Cypress integrates in CI quite easily, there are bunch of examples for most of CI providers. 
Best to run cypress as docker container, there are pre-build images or you can build it yourself from examples...

If you don't use Dashboboard service, you can publish cypress aftifacts and use any `mocha-reporter`, https://github.com/adamgruber/mochawesome is my favorite so far.

Watchout for cache folder, which cypress use for installation -  if you have multi-job pipeline, you have to pass it as cashed folder or artifact to upstream jobs.
With version 3.1.0 you can setup parallel runs with your CI runs.

### Services
Dashboard service from cypress allow you to look results of test runs. It have some history, video recordings and screenshots + test failure reasons in case of any. 
With new cypress 3.1.0, Dashboard service also behaves as scheduler/dispatcher for parallel runs of your tests. Before all tests runs were sequencial, which is not best for debugging, but now it's much improved.

### Other bells and rings

Cypress have quite cool **time travel** feature. It allow you to look back on state of application and DOM, which can be quite helpful for debug.

In general, debuging is one of strong sides of cypress, you can use regular `debugger()` inside your tests to break in test execution and add hook to Chrome debugger.

There is UI inspector that can help you to build actions and locators (if you need this kind of support)

Stubs, spies and clocks can help you to inject some state (if you work with Redux or some other state machine), simulate network responses and error handling (how your app behaives on 500 error)

Cypress waits for element to be on page automatically, so you don't need to make any async functions to support relative waits by yourself.

### What can be done better
What I really miss is a out-of-the-box rerunner for failed tests. Cypress is way less flacky as selenium,f.e. network lag is removed as cypress lives in same runtime of browser.
You can implement re-run using cypress Module API, but it's not as easy and native as I would like to see it, and you need to make a choice if you want parallel run or re-run.

Because of fundamental technical choises cypress team did, it's fairly impossible to run tests on non-chromium browsers. 

There are bunch of trade-offs you should be aware https://docs.cypress.io/guides/references/trade-offs.html#Temporary-trade-offs

## Summary

### Tools comparison

| Criteria         | Cypress    | TestCaffe | Puppitier |
| ---------------- | :--------: | --------: | --------: |
| Language support | JS         |           |           |
| Browser Support  | Chrome     |           |           |
| Pricing          | Free(beta) |           |           |



#Links
- https://docs.cypress.io/api/cypress-api/custom-commands.html#Best-Practices
- https://docs.cypress.io/faq/questions/using-cypress-faq.html
- https://docs.cypress.io/guides/core-concepts/writing-and-organizing-tests.html
- https://github.com/cypress-io/testing-workshop-cypress - nice workshop to follow full integration of cypress 
- https://github.com/cypress-io/cypress-example-recipes more than enought of recepies for get you started