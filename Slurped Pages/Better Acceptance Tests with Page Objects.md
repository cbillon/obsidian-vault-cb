---
link: https://thoughtbot.com/blog/better-acceptance-tests-with-page-objects
byline: Josh Clayton
site: thoughtbot
date: 2012-11-15T00:00
excerpt: Use a Page Object to improve acceptance readability.
twitter: https://twitter.com/@thoughtbot
slurped: 2025-03-12T09:58
title: Better Acceptance Tests with Page Objects
tags:
  - behat
  - test
---

During my Test-Driven Rails workshop earlier this week (which is also available as an [online workshop](https://thoughtbot.com/upcase/test-driven-rails)), my students and I were writing acceptance tests surrounding marking todo items as complete. The spec looked like this:

```
feature 'Manage todos' do
  scenario 'view only todos the user has created' do
    sign_in_as 'other@example.com'
    create_todo_titled 'Lay eggs'
    sign_in_as 'me@example.com'
    user_should_not_see_todo_titled 'Lay eggs'
  end

  scenario 'complete my todos' do
    sign_in_as 'person@example.com'
    create_todo_titled 'Buy eggs'
    complete_todo_titled 'Buy eggs'
    user_should_see_completed_todo_titled 'Buy eggs'
  end

  scenario 'mark my todos incomplete' do
    sign_in_as 'person@example.com'
    create_todo_titled 'Buy eggs'
    complete_todo_titled 'Buy eggs'
    mark_incomplete_todo_titled 'Buy eggs'
    user_should_see_incomplete_todo_titled 'Buy eggs'
  end

  def create_todo_titled(title)
    click_link 'Create a new todo'
    fill_in 'Title', with: title
    click_button 'Create'
  end

  def user_should_see_todo_titled(title)
    within 'ol.todos' do
      expect(page).to have_css 'li', text: title
    end
  end

  def user_should_see_completed_todo_titled(title)
    within 'ol.todos' do
      expect(page).to have_css 'li.complete', text: title
    end
  end

  def user_should_see_incomplete_todo_titled(title)
    within 'ol.todos' do
      expect(page).not_to have_css 'li.complete', text: title
    end
  end

  def user_should_not_see_todo_titled(title)
    within 'ol.todos' do
      expect(page).not_to have_css 'li', text: title
    end
  end

  def complete_todo_titled(title)
    todo = Todo.where(title: title).first
    within("[data-id='#{todo.id}']") { click_link 'Complete' }
  end

  def mark_incomplete_todo_titled(title)
    todo = Todo.where(title: title).first
    within("[data-id='#{todo.id}']") { click_link 'Incomplete' }
  end
end
```

There’s a handful of things that can probably be refactored in the helper methods, but that’s not what I wanted to focus on; check out the scenarios themselves:

```
scenario 'create a new todo' do
  sign_in_as 'person@example.com'
  create_todo_titled 'Buy eggs'
  user_should_see_todo_titled 'Buy eggs'
end

scenario 'view only todos the user has created' do
  sign_in_as 'other@example.com'
  create_todo_titled 'Lay eggs'
  sign_in_as 'me@example.com'
  user_should_not_see_todo_titled 'Lay eggs'
end

scenario 'complete my todos' do
  sign_in_as 'person@example.com'
  create_todo_titled 'Buy eggs'
  complete_todo_titled 'Buy eggs'
  user_should_see_completed_todo_titled 'Buy eggs'
end

scenario 'mark completed todo as incomplete' do
  sign_in_as 'person@example.com'
  create_todo_titled 'Buy eggs'
  complete_todo_titled 'Buy eggs'
  mark_incomplete_todo_titled 'Buy eggs'
  user_should_see_incomplete_todo_titled 'Buy eggs'
end
```

While each of these lines reads well, the subject of each line is the user, or “you”. _You_ sign in, _you_ create a todo titled “Buy eggs”, and _you_ should see a todo with the correct title; the focus should be the todo. The todo is the subject of the test; we’re making assertions about if it’s on the page and its state after certain page interactions occur. The other thing we’re doing is repeating the todo title everywhere, which seems too verbose.

What if we used a [Page Object](http://code.google.com/p/selenium/wiki/PageObjects)?

```
scenario 'create a new todo' do
  sign_in_as 'person@example.com'
  todo = todo_on_page
  todo.create

  expect(todo).to be_visible
end

scenario 'view only todos the user has created' do
  sign_in_as 'other@example.com'
  todo = todo_on_page
  todo.create

  sign_in_as 'me@example.com'

  expect(todo).not_to be_visible
end

scenario 'complete my todos' do
  sign_in_as 'person@example.com'
  todo = todo_on_page
  todo.create
  todo.mark_complete

  expect(todo).to be_complete
end

scenario 'mark completed todo as incomplete' do
  sign_in_as 'person@example.com'
  todo = todo_on_page
  todo.create
  todo.mark_complete
  todo.mark_incomplete

  expect(todo).not_to be_complete
end
```

Aside from signing in, everything focuses on the todo since it’s the star of the show. All that needs to be done is move the helper methods from the original code into methods `#create`, `#mark_complete`, `#mark_incomplete`, `#visible?`, and `#complete?`. First, we’ll start with `#todo_on_page`, though:

```
def todo_on_page
  TodoOnPage.new('Buy eggs')
end
```

An instance of `TodoOnPage`, instantiated with a specific title, is returned which will implement the handful of methods enumerated above.

```
class TodoOnPage < Struct.new(:title)
  include Capybara::DSL

  def create
    click_link 'Create a new todo'
    fill_in 'Title', with: title
    click_button 'Create'
  end

  def mark_complete
    todo_element.click_link 'Complete'
  end

  def mark_incomplete
    todo_element.click_link 'Incomplete'
  end

  def visible?
    todo_list.has_css? 'li', text: title
  end

  def complete?
    todo_list.has_css? 'li.complete', text: title
  end

  private

  def todo_element
    find 'li', text: title
  end

  def todo_list
    find 'ol.todos'
  end
end
```

By using well-named methods like `#todo_element` and `#todo_list`, it becomes immediately obvious how to mark todos complete and incomplete, as well as how to check if a todo is complete or on the page.

The `TodoOnPage` is a page object. Given a specific context (in this case, the title of a todo), it encapsulates page interaction (`#create`, `#mark_complete`, `#mark_incomplete`) and assertions (using RSpec’s predicate matchers with `#visible?` and `#complete?`).

While I’ve written plenty of page objects before, often they only involve interaction and not predicate methods for matchers. Including both seems totally obvious now; I’m really excited to start using this pattern throughout other areas of my acceptance testing.
