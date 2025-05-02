---
link: https://dev.to/hecrj/use-opaque-types-in-elm-3oal
byline: Héctor Ramón
site: DEV Community
date: 2018-10-27T19:40
excerpt: In this article we will learn how opaque types can glue an API together while making our code self-documenting and bug-free.
twitter: https://twitter.com/@
tags:
  - elm
  - type
slurped: 2024-08-17T18:35
title: Use opaque types in Elm!
---

It's been a while since I joined the [Elm Slack](https://elmlang.herokuapp.com/). During this time, I have seen different folks ask how, and when, to use **opaque types**. I have also seen many codebases and examples where opaque types were non-existent.
I think opaque types are extremely important when it comes to API design in Elm and I suspect they are not used nearly enough. In this article we will learn how opaque types can **glue** an API together while making our code **self-documenting** and **bug-free**.

## [](https://dev.to/hecrj/use-opaque-types-in-elm-3oal#a-common-scenario)A common scenario

We will work with an example of what I think is a fairly common scenario: a form that validates and submits some data to create a new resource remotely. This kind of interaction is the core of many interactive web applications.

For simplicity, we will assume the resource we want to create is a very simple **question** with a title and a body.

First, we will start with code that does not use any opaque types. Then, we will be gradually adding them and analyzing how code has improved along the way!

Let's begin!

## [](https://dev.to/hecrj/use-opaque-types-in-elm-3oal#an-example-with-primitives)An example with primitives

We want to create a **question**. Let's say we write a module `Question` with this API:  

```
module Question exposing (create)

create : String -> String -> Task Http.Error String
```

This `create` function takes a question `title` and a `body` and returns a `Task` which tries to create the appropriate question. This `Task` can either fail and return an `Http.Error`, or succeed and return a `String` representing the `slug` of the brand new question.

Then, we build a form so our users can create questions. We will focus on the code that submits the form, leaving out some irrelevant details like how the fields are updated/rendered:  

```
type alias Model =
    { title : String
    , body : String
--  , ...
    }

type Msg
    = Submit
    | QuestionCreated (Result Http.Error String)
--  | ...

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        Submit ->
            ( model
            , Question.create model.title model.body
                |> Task.attempt QuestionCreated
            )

        -- ...

view : Model -> Html Msg
view model =
    form
      [ onSubmit Submit ]
      [ titleField model
      , bodyField model
      , submitButton model
      ]
```

Do you notice any issues here? Let's see. What happens when we `Submit` our form? We create the question using `Question.create`. Sounds good! But wait... Where is validation?! What happens if `model.title` is empty? What if `model.body` is extremely long? Hmm... Okay, no problem. We just need to add an if-then-else, right?  

```
case msg of
    Submit ->
        if Question.titleIsValid model.title && Question.bodyIsValid model.body then
            ( model
            , Question.create model.title model.body
                |> Task.attempt QuestionCreated
            )
        else
            ( model, Cmd.none )

    -- ...
```

Cool! We are done here. The new `titleIsValid` and `bodyIsValid` functions deal with validation and there is no way to create an invalid question now.

However, doesn't this feel a bit off? Whenever we want to use `create` we have to **remember** to validate its input by using `titleIsValid` and `bodyIsValid`. What if some requirements change and `create` needs another argument? We will have to **remember** to update the if-then-else. What if some new developer reads the type signature of `create` and decides to use it somewhere else without any kind of validation? Not desirable.

In the end, our API is error-prone. **It can be used incorrectly**. Can we do better? Wouldn't it be nice if we could somehow tie the concept of validating and creating a question together?

## [](https://dev.to/hecrj/use-opaque-types-in-elm-3oal#opaque-types-to-the-rescue)Opaque types to the rescue!

Before we continue, let's review what an opaque type is:

> An opaque type is a custom type without exposed constructors.

Okay, but if the constructors are not exposed... How do we use an opaque type? Well, the constructors are accessible internally, in the module where the custom type has been defined. Therefore, we can expose functions to _control_ how the values of the opaque type are created.

Let's see how this works! Going back to our `Question` module:  

```
module Question exposing (create)

create : String -> String -> Task Http.Error String
```

The issue with `create` is that the types of its arguments accept invalid domain values. For instance, `""` is a perfectly valid `String` but it is not a valid question title.

Can we create a new custom type where its possible values are always valid question titles? Let's see:  

```
module Question exposing (Title, create, titleFromString)

type Title
    = Title String

titleFromString : String -> Result String Title
titleFromString title =
    if String.length title < 5 then
        Err "the title must not be less than 5 characters long"
    else if String.length title > 100 then
        Err "the title must not be more than 100 characters long"
    else
        Ok (Title title)

-- ...
```

That's it! Here we define a new custom type `Title` without exposing its constructor. Instead, we implement a `titleFromString` function which takes a `String` and returns either a `String` describing an error when validation fails, or a `Title` when validation succeeds.

The only way to build a `Title` is to use `titleFromString`. As a consequence, if we have a `Title` anywhere in our codebase, we can be confident that it is a valid question title. The `Title` type **guarantees validity**.

Whenever we are validating data, we are just checking that the data has some **guarantees**. Performing validation and then using the same type afterwards is a missed opportunity! We should try to capture these guarantees using opaque types. As a result, our APIs will become safer and easier to understand.

Similarly, we can define a `Body` type. Then, we can update the `create` function:  

```
module Question exposing (Body, Title, create, bodyFromString, titleFromString)

-- type Title ...
-- type Body ...

create : Title -> Body -> Task Http.Error String
```

Now, the `create` function forces us to provide a valid title and a valid body. We cannot longer submit the form as we did before. We must use `titleFromString` and `bodyFromString`:  

```
case msg of
    Submit ->
        case ( Question.titleFromString model.title, Question.bodyFromString model.body ) of
            ( Ok title, Ok body ) ->
                -- Validation succeded
                ( model
                , Question.create title body
                    |> Task.attempt QuestionCreated
                )

            _ ->
                -- Validation failed
                ( model, Cmd.none )

    -- ...
```

Our API does not provide any other way to do this. We cannot skip validation anymore! We do not even need to **remember** validation. The API **forces** us to deal with errors along the way. Our API is **safer**.

## [](https://dev.to/hecrj/use-opaque-types-in-elm-3oal#further-improvements)Further improvements

We are not done yet! There are still a couple of things we can improve.

The first improvement has to do with **duplicated validation**. We are currently performing validation in our `update` function when the form is submitted, but we probably want to validate the form fields in our `view` code too, maybe show error messages in real time. Therefore, `update` and `view` are both using `titleFromString` and `bodyFromString`: `update` only cares about the successful result, while `view` only cares about the errors.

Now that we have `Title` and `Body` types, we can make our `Submit` message **impossible to be fired if the form values are invalid**, validating only once in `view` and propagating the **validation guarantees** over to `update`. We just need to change the `Submit` message:  

```
type Msg
    = Submit Question.Title Question.Body
    -- |...


case msg of
    Submit title body ->
        ( model
        , Question.create title body
            |> Task.attempt QuestionCreated
        )

    -- ...
```

Clean! We got rid of the awkward no-operation in the else clause when validation failed.

This particular approach, where form validation happens in `view` while capturing guarantees using opaque types, is one of the main ideas behind a package that I wrote: [`composable-form`](https://github.com/hecrj/composable-form). [`composable-form`](https://github.com/hecrj/composable-form) treats forms as composable units, so they can be built, combined, and reused freely. I will write about it soon!

The second improvement is simple. We can create a `Slug` opaque type. Then, the `create` function will look even better:  

```
create : Title -> Body -> Task Http.Error Slug
```

Now, if we ever need to allow our users to edit questions, we could just write an `edit` function:  

```
edit : Slug -> Title -> Body -> Task Http.Error Slug
```

Neat! Our API will explicitly say a question can only be edited if it has been previously created.

## [](https://dev.to/hecrj/use-opaque-types-in-elm-3oal#in-summary)In summary

Opaque types not only make your code safer, but they can also be used to connect different concepts together, making your codebase much **easier to understand**.

Here is one exercise I like to perform when I code: I try to imagine the thought process that a new developer, with a specific goal in mind, will have when reading the module documentation. Let's do that with our `Question` module, assuming Bob wants to find out how to `create` a new question:

1. Bob finds a `create` function.
2. Bob sees he can `create` a question if he provides a `Title` and a `Body`.
3. Bob wonders if there is some way to build these using a `String`.
4. Bob finds `titleFromString` and `bodyFromString`, which seem to do the job.
5. Bob notices those functions return a `Result`.
6. Bob understands he will need to handle the case where the provided `String` is not valid.

In this case, the developer can get a clear understanding of the `Question` API just by looking at the type signatures. No comments needed!

---

