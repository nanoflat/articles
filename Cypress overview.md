- [Architecture](#architecture)
- [Usage](#usage)
    - [Cypress configuration](#cypress-configuration)
    - [Test framework design with Cypress](#test-framework-design-with-cypress)
    - [Cypress in CI](#cypress-in-ci)
- [Cypress Dashboard Service](#cypress-dashboard-service)

### Architecture

Cypress main parts are:
- Runner - chrome app, glue between the driver, reporter, server, and extension 
- Driver - the library that loads inside of your browser and executes commands in runtime
- Extension - chrome extension
- Server - node app behind the browser, responsible for all communication and external processes (reporters, plugins, other node processes)
- Reporter - UI component for showing process and test results

### Usage

#### Cypress configuration
Cypress setup is defined in `cypress.json`. There you can set up base URL of your page, timeouts, including and excluding policies, video recordings and a bunch of other settings. Same you can set up in .env.json files and as ENV variables or run parameters for cypress command line.

`support/defaults.js` - for overwriting defaults of cypress. F.e. I use it for whitelisting some cookies that I persist between runs

`support/index.js` - global config and behaviour of cypress. F.e. there I overwrite `before()` method, as I need to have it by default before all tests

`support/assertions.js`  - for extending assertions that you can use in your tests. F.e. I extend it with `chai-almost` assertion.

`support/commands.js` - for extending cypress commands. F.e. you can extend cypress with your custom `cy.login()` command for some common actions you execute. I don't use it as I prefer to have my own commands and methods rather than extending cypress. 

`support/index.js` - for your custom plugins. It's executed when you (re)open project. I use it to load cypress configs for different environments.

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