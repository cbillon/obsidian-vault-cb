---
link: https://thoughtbot.com/blog/four-phase-test
byline: Dan Croak
site: thoughtbot
date: 2012-09-28T00:00
excerpt: |-
  The Four-Phase Test is a
  testing pattern, applicable to all programming...
twitter: https://twitter.com/@thoughtbot
slurped: 2025-03-12T09:46
title: Four-Phase Test
tags:
  - behat
  - test
---

The [Four-Phase Test](http://xunitpatterns.com/Four%20Phase%20Test.html) is a testing pattern, applicable to all programming languages and unit tests (not so much integration tests).

It takes the following general form:

```
test do
  setup
  exercise
  verify
  teardown
end
```

There are four distinct phases of the test. They are executed sequentially.

## [setup](https://thoughtbot.com/blog/four-phase-test#setup)

During setup, the system under test (usually a class, object, or method) is set up.

```
user = User.new(password: 'password')
```

## [exercise](https://thoughtbot.com/blog/four-phase-test#exercise)

During exercise, the system under test is executed.

## [verify](https://thoughtbot.com/blog/four-phase-test#verify)

During verification, the result of the exercise is verified against the developer’s expectations.

```
user.encrypted_password.should_not be_nil
```

## [teardown](https://thoughtbot.com/blog/four-phase-test#teardown)

During teardown, the system under test is reset to its pre-setup state.

This is usually handled implicitly by the language (releasing memory) or test framework (running inside a database transaction).

## [all together](https://thoughtbot.com/blog/four-phase-test#all-together)

The four phases are wrapped into a named test to be referenced individually.

Our [style guide](https://github.com/thoughtbot/guides/tree/master/style) advises we “Separate setup, exercise, verification, and teardown phases with newlines.”

```
it 'encrypts the password' do
  user = User.new(password: 'password')

  user.save

  user.encrypted_password.should_not be_nil
end
```

Go forth and test in phases.
There are four distinct phases of the test. They are executed sequentially.

## [setup](https://thoughtbot.com/blog/four-phase-test#setup)

During setup, the system under test (usually a class, object, or method) is set up.

```
user = User.new(password: 'password')
```

## [exercise](https://thoughtbot.com/blog/four-phase-test#exercise)

During exercise, the system under test is executed.

```
user.save
```

## [verify](https://thoughtbot.com/blog/four-phase-test#verify)

During verification, the result of the exercise is verified against the developer’s expectations.

```
user.encrypted_password.should_not be_nil
```

## [teardown](https://thoughtbot.com/blog/four-phase-test#teardown)

During teardown, the system under test is reset to its pre-setup state.

This is usually handled implicitly by the language (releasing memory) or test framework (running inside a database transaction).

## [all together](https://thoughtbot.com/blog/four-phase-test#all-together)

The four phases are wrapped into a named test to be referenced individually.

Our [style guide](https://github.com/thoughtbot/guides/tree/master/style) advises we “Separate setup, exercise, verification, and teardown phases with newlines.”

```
it 'encrypts the password' do
  user = User.new(password: 'password')

  user.save

  user.encrypted_password.should_not be_nil
end
```

Go forth and test in phases