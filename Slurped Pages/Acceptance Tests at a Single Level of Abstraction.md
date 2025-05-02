---
link: https://thoughtbot.com/blog/acceptance-tests-at-a-single-level-of-abstraction
byline: Josh Clayton
site: thoughtbot
date: 2014-09-02T00:00
excerpt: "Each acceptance test tells a story: a logical progression through a task..."
twitter: https://twitter.com/@thoughtbot
slurped: 2025-03-12T09:44
title: Acceptance Tests at a Single Level of Abstraction
tags:
  - behat
  - test
---

Each acceptance test tells a story: a logical progression through a task within an application. As developers, it’s our responsibility to tell each story in a concise manner and to keep the reader (either other developers or our future selves) engaged, aware of what is happening, and understanding of the _purpose_ of the story.

At the heart of understanding the story being told is a consistent, single level of abstraction; that is, each piece of behavior is roughly similar in terms of functionality extracted and its overall purpose.

## [An example of multiple levels of abstraction](https://thoughtbot.com/blog/acceptance-tests-at-a-single-level-of-abstraction#an-example-of-multiple-levels-of-abstraction)

Let’s first focus on an example of how _not_ to write an acceptance test by writing a test at multiple levels of abstraction.

```
# spec/features/user_marks_todo_complete_spec.rb
feature "User marks todo complete" do
  scenario "updates todo as completed" do
    sign_in
    create_todo "Buy milk"

    find(".todos li", text: "Buy milk").click_on "Mark complete"

    expect(page).to have_css(".todos li.completed", text: "Buy milk")
  end

  def create_todo(name)
    click_on "Add new todo"
    fill_in "Name", with: name
    click_on "Submit"
  end
end
```

Let’s focus on the scenario. We’ve followed the [four-phase test](https://thoughtbot.com/blog/four-phase-test), separating each step:

```
scenario "updates todo as completed" do
  # setup
  sign_in
  create_todo "Buy milk"

  # exercise
  find(".todos li", text: "Buy milk").click_on "Mark complete"

  # verify
  expect(page).to have_css(".todos li.completed", text: "Buy milk")

  # teardown not needed
end
```

To prepare for testing that marking a todo complete works, we sign in and create a todo to mark complete. Once the todo is complete, we find it on the page and click the ‘Mark complete’ anchor tag associated with it. Finally, we ensure the same `<li>` is present, this time with a completed class.

From a behavior standpoint, this touches on each part of the app necessary to verify marking a todo as complete works; however, there are varying levels of abstraction in this test between the setup phase and exercise/verify phases. There are Capybara methods (`find` and `have_css`) interspersed with helper methods (`sign_in` and `create_todo`) which force developers to switch from user-level needs and outcomes to page-specific interactions like checking for presence of specific elements with CSS selectors.

## [An example of a single level of abstraction](https://thoughtbot.com/blog/acceptance-tests-at-a-single-level-of-abstraction#an-example-of-a-single-level-of-abstraction)

Let’s now look at a scenario written at a single level of abstraction:

```
feature "User marks todo complete" do
  scenario "updates todo as completed" do
    sign_in
    create_todo "Buy milk"

    mark_complete "Buy milk"

    expect(page).to have_completed_todo "Buy milk"
  end

  def create_todo(name)
    click_on "Add new todo"
    fill_in "Name", with: name
    click_on "Submit"
  end

  def mark_complete(name)
    find(".todos li", text: name).click_on "Mark complete"
  end

  def have_completed_todo(name)
    have_css(".todos li.completed", text: name)
  end
end
```

This spec follows the [Composed Method pattern](http://c2.com/ppr/wiki/WikiPagesAboutRefactoring/ComposedMethod.html), discussed in [Smalltalk Best Practice Patterns](http://www.amazon.com/Smalltalk-Best-Practice-Patterns-Kent/dp/013476904X), wherein each piece of functionality is extracted to well-named methods. Each method should be written at a single level of abstraction.

While we’re still following the four-phase test, the clarity provided by reducing the number of abstractions is obvious. There’s largely no context-switching as a developer reads the test because there’s no interspersion of Capybara helper methods with our methods (`sign_in`, `create_todo` `mark_complete`, and `have_completed_todo`).

The most common ways to introduce a single level of abstraction are to extract behavior to helper methods (either within the spec or to a separate file if the behavior is used across the suite) or to extract [page objects](https://thoughtbot.com/blog/better-acceptance-tests-with-page-objects).

The cost of going down the path of high-level helpers across a suite isn’t nonexistent, however; by extracting behavior to files outside the spec (especially as the suite grows and similar patterns emerge), the page interactions are separated from the tests themselves, which reduces cohesion.

Maintaining a single level of abstraction is a tool in every developer’s arsenal to help achieve clear, understandable tests. By extracting behavior to well-named methods, the developer can better tell the story of each scenario by describing behaviors consistently and at a high enough level that others will understand the goal and outcomes of the test.