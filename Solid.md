---
tags:
  - solid
  - interface
  - principles
---
_Let's be honest, writing good code is not easy. Writing code that everybody can understand, code that is easy to test and navigate through, is **hard**. If you're hyped about making your code better, but you are just starting out with SOLID principles, design and code patterns, it can get confusing just to start writing the code. How to create a class with a single responsibility, how to KISS, when YAGNI or to DRY, etc., are just some of the questions that pop into your mind. Take a step back, take a deep breath, and just start coding._

Now, I'm not saying you shouldn't think about the design, structure, and code quality of your project. You should keep these in mind **all the time** while you are working on your code. I'm saying that it is hard to pre-plan everything and anticipate all of the requirements for your service or your app. Especially if some of those requirements are not known when you are starting out with your project. Even then, changes in your requirements can throw a monkey wrench in your design if it's not flexible enough and you will need a refactoring effort to some extent. So, just start coding and keep in mind that change is inevitable. The code that you write today will probably need to be changed sometimes in the future if changes affect it.

The most undersold point of SOLID principles is that the state of your code, or just part of your code, should be reviewed constantly, all through the lifecycle of your project. Sure, SOLID helps with readability, easier and faster development and testing, improves code quality, etc. But, because the code in your project is always evolving, it's quality and flexibility can deteriorate if it is constantly not reviewed, maintained, and improved.

### [](https://lalatron.hashnode.dev/immediately-start-improving-your-code-with-isp#but-what-about-the-interface-segregation-principle "Permalink")But what about the Interface Segregation principle?

Patterns emerge and are easier to spot when you actually have some code to look at. Way more easier than to anticipate the shape of your classes, models, repositories, and layers at the beginning. The Interface Segregation Principle is one of those principles that is very hard to anticipate when starting out with designing your app or your service. It is much much easier to adhere to this principle when you actually have some code to look at. And, if you have an approach of always reviewing and improving your code, it can be one of the easiest principles to implement. Consider the example below, written in Go.

You have created an app that shows different posts by users that use that same app. You are rolling out features in iterations and the first iteration is to build authentication and enable the user to create, update, and delete a post, and also see all of its posts on its personal page. So, you design and create a layered REST API service and a main `PostRepository` interface that looks something like this:

Copy

Copy

```
type PostRepository interface {
    Get(id uint64) (Post, error)
    GetAllByUserID(userID uint64) ([]Post, error)
    Update(post *Post) error
    Create(post *Post) error
    Delete(id uint64) error
}


// MeController is used to handle all of user personal actions
type MeController struct {
    Posts PostRepository
}
```

This is just fine for the first iteration since all the management and querying of users' posts come from a single point of entry that is the user's personal page. The `PostRepository` is used in the `MeController`, the only place where a user is available to create, edit, and delete its posts.

Here come the next iterations and additional developers to help out with your REST API service. New requirements are to build a main page where the user can see all posts from the people he follows and to see a list of posts from a user that is followed.

Since the new requirements are related to posts, it might be tempting to put new methods in the`PostRepository`, but there are many good reasons not to do that. One that we will focus on is to restrict the access to methods that manage posts (create, edit, delete). This is where the Interface Segregation Principle comes into play.

> The interface-segregation principle (ISP) states that no client should be forced to depend on methods it does not use.

In our example, the purpose of the Interface Segregation Principle is to restrict component access from the parts of the code that those components don't use and should not use. The design decision here is that the `Create`, `Update`, and `Delete` methods should only be allowed in the`MeController` struct. The refactoring effort is as easy as creating more repository interfaces and rearranging the methods between these interfaces. It might not be the best approach, but it is good enough for this iteration. See the code below.

_Please note that the design above is here just to explain the Interface Segregation Principle. It can be improved even more by considering more principles, but for the time being, we are focused on this one._

Copy

Copy

```
type PostManagerRepository interface {
    Update(post *Post) error
    Create(post *Post) error
    Delete(id uint64) error
}

type UserPostReaderRepository interface {
    Get(id uint64) (Post, error)
    GetAllByUserID(userID uint64) ([]Post, error)
}

type AllPostsReaderRepository interface {
    GetAllPostsFromLinkedUsers(userID uint64) ([]Post, error)
}

// MeController is used to handle all of user personal actions like create, update and delete of a post
type MeController struct {
    PostManager PostManagerRepository
    Posts       UserPostReaderRepository
}

// MainController is used to handle actions from the main feed
type MainController struct {
    Posts AllPostsReaderRepository
}

// User controller is used to handle actions when viewing posts for another specific user
type UserController struct {
    Posts UserPostReaderRepository
}
```

### [](https://lalatron.hashnode.dev/immediately-start-improving-your-code-with-isp#conclusion "Permalink")Conclusion

The purpose of this article is to show that overthinking is not necessary when trying to design or refactor your code. A good approach is to do it gradually and often. In the case of the Interface Segregation Principle, it is as simple as going through your code, deciding what needs restricted access, what can be logically grouped together, and making the changes. That is good enough for the first step. Just be sure that you plan the next steps as well. Your code is evolving constantly, and with it, you should too.