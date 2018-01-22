---
layout: post
title: the problem with cukes
date: 2018-01-21
tags: development, testing
---

One of my first tasks as a software developer (almost two years ago!) was to start writing [cucumber](cucumber.io) tests for a project called [Hootenanny](https://github.com/ngageoint/hootenanny), a high performance spatial data conflation software. In retrospect, this was a great initial task considering

1. I had never written any kind of test before. Ever.
2. I was still at the stage in learning JavaScript where you console log everything because you don't know what it is (but I still do that...)

Given the test environment was already set up (more on that later...), just writing cucumber tests actually became really enjoyable and a great learning tool. Firstly, writing tests is pretty easy. Cucumber tests are intended to simulate user-client interaction, written with very plain-language step outlines controlled by a more intricate set of step definitions. You learn how dumb computers are and write things to force the browser to click buttons. It's pretty awesome and I had a knack for writing tests. Our test suite did a pretty good job of unveiling bugs. Cue my infatuation with cucumber.

I ended up setting up test suites with cucumber for a couple of our other projects and deployed them in Jenkins. Then things started getting weird and tests would randomly and intermittently fail, even though they passed locally. Eventually, this led me to do some research into why you _shouldn't_ use cucumber, and some of the problems with our test suite. This is what I gathered:

### :cucumber: The first step of writing a UI test: ask yourself if you really need to write a UI test
Unit testing is great, and now I am a believer that code should be written based on TDD, or else you're probably writing shitty code. Make sure you're getting expected outputs from all your functions, then test UI interaction after.

### :cucumber:  Parallel environments can be tricky.

Most of our projects used [vagrant](vagrant) to develop on locally as well as deploy a workspace in Jenkins. We had some discrepancies with boxes (vSphere vs. ubuntu/trusty64 base box, ubuntu vs. vagrant being the sudoer, etc., _don't ask me why... just learn from my mistakes!_) that lead to ownership issues, like sometimes tests would fail because chromedriver couldn't be accessed by the user executing the test commands. For one project, the virtual environment got rebuilt nightly, and gems and other dependencies like chromedriver would get updated (_always lock your versions!_) because we didn't lock any dependency versions. Cucumber depends on quite a few gems (capbyara, rspec, selenium webdriver, etc. as well as non-gems) so there are a lot of unknowns when tests fail for no reason and you didn't lock your gem versions.

### :cucumber: Tests can (and should!) be written to be reusable.
#### _Which in the case of cukes, is a bad thing..._

You can use css selectors and text in tests as matchers, which is cool because you can write a simple test step like this:

```
When(/^I click the "([^"]*)" link$/) do |linkText|
  find('a', :text=> linkText).click
end
```

Or a seemingly harmless one like this:

```
When(/^I select the "([^"]*)" option in "([^"]*)"$/) do |opt, el|
  combobox = page.find(el)
  combobox.find('.combobox-caret').click
  page.find('div.combobox').find('a', :text=> opt).click
end
```

Which leads to ugly feature steps like:

```
And I select the "true" option in "#containerofisCollectStats"
```

The first one is clear cut, and can most likely be re-used for any link. It is easy to recognize out to re-use. The second one is verbose, uses 2 unique matchers, is written in a way that it can be re-used but is so specific that it doesn't. In fact, we had another step definition in the same suite that was very similar:

```
Then(/^I click on the "([^"]*)" option in the "([^"]*)"$/) do |label,div|
  find('#' + div).find('label', :text => label).click
end
```

When collaborating on cukes, sometimes it's hard to recognize how steps can be reused, so people end up writing their own step definitions, and it causes repetition and headache (and heartache... _why didn't you use my cukes?!_), and ultimately a really long step definition file.

### :cucumber: Organize your step definitions

Another solution to the above might be to organize your step definition differently. Each project I've worked on had a single `custom_steps.rb` file under `step_defintions`. I think in the future, I might organize my step definitions by test and break them into separate files since cucumber reads everything in your `step_definitions` folder.

### :cucumber: Don't write tests that depend on external variables to pass

One of our projects had a separate geocoding service that was used to test certain UI elements. This test would fail whenever the geocoder was down, even if nothing was wrong with the code. I suppose this was good to know, but it also slowed down development since we would hold PRs until all tests passed.

### :cucumber: It's low level

[Cucumber is intended for non-developers](https://8thlight.com/blog/kevin-liddle/2013/09/18/a-case-against-cucumber.html), which may be the reason why I liked it so much when I was first learning how to be a developer. Features are not test scripts. If developers are the ones reading and writing your UI tests, why use something as disjointed and menial as cucumber? Can you just write a capybara/rspec script that ultimately manipulates the browser in the same way?

### :cucumber: Avoid conjunctive steps

Conjunctive steps make it difficult to identify where a test failed if it fails. See [cucumber best practices](https://blog.codeship.com/cucumber-best-practices/) for examples.

:cucumber:
:cucumber:
:cucumber:
:cucumber:

Like I said previously, learn from what I now know. Cucumber+Capybara is kind of fun to write, and I enjoyed being able to tell Jenkins to run tests with emojis, but in our case, it caused a lot of headache and heartache, and you should probably be focusing more on your unit tests.


![](../../../../img/cucumber-cat.gif)
