---
tags:
  - php
  - tools
---

====== Outils Qualité du Code ======

[[https://phpqa.io/| Outils décrits dabs cet article]]

Long-term projects are hard to maintain and deal with but these code quality tools can help you out if you use them consistently.

As someone who had a chance to work on very complex line of business applications with complex business rules, I've noticed lack of code quality due to different reasons which may include:

  * Inexperience
  * Time constraints
  * and many more reasons

Over the years, I've noticed that there are a lot of not-so-known code quality tools and ways that can help me improve my own code and keep it maintainable, which is very hard, especially when the project grows in terms of business complexity after years of it being developed.

Here's a list of code quality tools and ways that I would recommend to people to use in order for them to become better software engineers as well as keep their code maintainable, and write less buggy code.

===== 1. PHP CS Fixer =====

This tool scans your project for potential coding style violations and it fixes them based on the configuration it has.

One of the recommended coding styles in the PHP community used to be the PSR-12 coding style guide which is now deprecated but it is being replaced with PER-CS.

You can write your own rules or customize the rules however your want.

The most important thing is to keep your and your team's code consistent.

You can run it manually to fix your code but I would recommend running it as part of your continuous integration pipeline so that you don't have to worry about such problems.

===== 2. PHPStan ====

Since PHP is not a strongly typed language and it does not compile before it runs like C# or Java, there's no way to know if your code is executing correctly until you run it manually.

In order to skip executing the code after every change, you can use a static analysis tool like PHPStan to help with this, and help you spot bugs before you deploy the code to production.

PHPStan is like TypeScript in the PHP world, which also allows you to write your own rules and modify the existing ones to fit your and your team's needs.

It may be weird at first when you start using it, because you are used to writing code and running it manually, not having to worry about some of the checks that this tool runs, but it does help you with keeping your code at a higher quality.

An alternative static analysis tool for PHPStan is Psalm which does similar checks and has similar rules to PHPStan, but they can also be used together wherever it makes sense.

You can run it manually to analyze your code but I would recommend running it as part of your continuous integration pipeline so that you don't have to worry about such problems.

===== 3. PHPArkitect ===

Whenever you are working on a bigger project that has to be maintained for years, it will require you to design and implement a specific architecture so that it's simple and easy to stay consistent with code changes due to the complex business rules and processes.

PHPArkitect is a tool that will help you stay consistent with your project folder / namespace structure.

It doesn't matter if you use Clean Architecture, Hexagonal Architecture, Vertical Slices or something custom, there are rules that you can set so that whenever you or your colleague creates a class in a wrong namespace or adds a controller as a dependency to another controller or an entity, this tool will catch it and let you know.

You can run it manually to analyze your code but I would recommend running it as part of your continuous integration pipeline so that you don't have to worry about such problems.

===== 4. Automated Testing ==

Writing different types of automated tests will help you cover as much code as possible and find bugs as well as refactor existing code but most importantly, it will help you drive the design of the feature implementations if you are using test-driven development.

Manual testing helps but it does not bring the value as much as automated tests if they are implemented properly.

Also, make sure to cover as many scenarios and as much implementation code as possible with the tests, otherwise they may not be as beneficial and they may become a waste of time after a while.

There are multiple test runners and frameworks in the PHP ecosystem that are worth looking at:

  * PHPUnit
  * PHPSpec
  * Behat
  * Codeception

They all do similar or different types of testing depending on what you are trying to accomplish and what approach you take.

You can run the tests manually to check your code if it works as expected but I would recommend running it as part of your continuous integration pipeline so that you don't have to worry about such problems.

===== 5. Infection ==

This tool will add an additional layer of scenarios for your automated tests to see if your tests cover all of the possible scenarios by mutating the code in small ways to see if the tests will fail or pass. If your tests fail, that's a good thing, but if your tests pass with the mutated implementation, that means you missed a test scenario and you may want to add it.

You can run the tests manually to check your code if it works as expected but I would recommend running it as part of your continuous integration pipeline so that you don't have to worry about such problems.

===== 6. Deptrac PHP ==

Deptrac is a static code analysis tool that analyses your code and checks if your code is written with decoupling in mind, where all of your packages / modules / extensions are working independently of each other.

You can run it manually to analyze your code but I would recommend running it as part of your continuous integration pipeline so that you don't have to worry about such problems.

===== 7. Rector ==

When you are working on long term projects, most of your focus is on the bug fixing and code refactoring which is usually considered maintenance compared to new feature implementations, which means you will need to focus on changing code all over the place and it will take time to do all changes manually.

Besides usual code improvements, you will also have to deal with upgrades to PHP as well as the packages that the project depends on.

Rector will help you refactor your code by specifically defined rules which will speed up your work for some pieces of code.

===== 8. Design patterns and principles ==

Whenever people plan the products that they need to implement based on the business needs, they usually plan the product for the short term rather than the long term, so design patterns and principles are usually misunderstood or misused so that's why they are considered over-engineering or complex.

Product features grow over the years and the code complexity rises so it will be way harder to change the code later on if the things are not implemented or planned for the long term.

Work on long term projects, make sure to learn about the different principles and design patterns that exist, what are their pros and cons, learn to identify the context of the project and if it makes sense to use that approach and experiment with the way that you write code in order to get used to write code for the long term.

Some of the most popular principles are the SOLID principles, which help you write framework agnostic code, where the framework that you love and use will just be a tool and aid the business rules rather than your business related code to depend on the rules of the framework.

Also, check out the idea of using composition over inheritance and its benefits.

Conclusion
All of these tools and ideas take time, planning and a lot more brain power to get used to using on a daily basis and stay consistent with them but it will save you a lot of trouble along the way and they will save money for the business. It may require more time upfront but that time will be cut down later on when the problems come up and the things get serious.

You may not see the benefits right away, or even see the point in using these ideas an tools if you don't have the experience working on a bigger project which has lasted at least a few years, but trust me, they were invented for a reason and they don't exist just because someone wants to complicate their own code and architecture for the sake of it (although that may be case with people who misuse or misunderstood these tools and ideas).

You can find more of these tools at PHPQA.io and see if they can help you out for your specific case.
