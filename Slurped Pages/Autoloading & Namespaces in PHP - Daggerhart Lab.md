---
link: https://www.daggerhartlab.com/autoloading-namespaces-in-php/
byline: Jonathan Daggerhart
site: Daggerhart Lab
date: 2022-05-16T19:39
excerpt: What are namespaces and autoloading in PHP? This post explains the What, Why, and How of this fundamental OOP concept, along with PSR-4.
twitter: https://twitter.com/@daggerhart
slurped: 2026-06-26T18:59
title: Autoloading & Namespaces in PHP - Daggerhart Lab
tags:
  - php
---

This post wil help you understand the What, Why, and How of autoloading, namespaces, and the `use` keyword in PHP.

If you’re already familiar with the What and Why of autoloading and want to skip right to the How, check out this post on [Composer Autoloader: The Easiest Autoloading in PHP](https://www.daggerhartlab.com/composer-autoloader-the-easiest-autoloading-in-php/).

Before we get into PHP Namespaces, let’s look at Autoloading.

## What is Autoloading?

Autoloading is a way to have PHP automatically include the PHP class files of a project.

Consider an OOP PHP project that has more than a hundred PHP classes. How might we make sure that all your classes are loaded before using them? Without autoloading, we might end up manually including every class, like this:

```
<?php// manually load every file the whole project usesrequire_once __DIR__.'/includes/SomeClass1.php';require_once __DIR__.'/includes/SomeClass2.php';require_once __DIR__.'/includes/SomeClass3.php';require_once __DIR__.'/includes/SomeClass4.php';require_once __DIR__.'/includes/SomeClass5.php';require_once __DIR__.'/includes/SomeClass6.php';require_once __DIR__.'/includes/SomeClass7.php';require_once __DIR__.'/includes/SomeClass8.php';require_once __DIR__.'/includes/SomeClass9.php';require_once __DIR__.'/includes/SomeClass10.php';// ... etc, a hundred more times$my_example = new SomeClass1();
```

This is a bit tedious to say the least, and unmaintainable at worst.

What if instead, we could have PHP automatically load class files when we need it? Spoiler alert, we can, and it is called “Autoloading”.

### PHP Autoloading 101

To create an autoloader only takes two steps:

1. Write a function that looks for files that need to be included
2. Register that function with the `spl_autoload_register()` core PHP function.

Let’s do this for the above example.

```
<?php/** * Simple autoloader * * @param $class_name - String name for the class that is trying to be loaded. */function my_custom_autoloader( $class_name ){    $file = __DIR__.'/includes/'.$class_name.'.php';    if ( file_exists($file) ) {        require_once $file;    }}// add a new autoloader by passing a callable into spl_autoload_register()spl_autoload_register( 'my_custom_autoloader' );$my_example = new SomeClass1(); // this works!
```

There we go. We have no longer need to manually `require_once` every single class file in the project. Instead, with our autoloader, the system will automatically require the files as their classes are used. For a better understanding of what is going on here, let’s walk through the exact steps in the above code.

1. We wrote a function named `my_custom_autoloader` which expects 1 parameter that we have called `$class_name`. Given a class name, the function looks for a file with that name, and loads that file if found.
2. `spl_autoload_register()` is a function in PHP that expects 1 “callable” parameter. A “callable” parameter can be many things, such as a function name, class method, or even an anonymous function. In our case, we provided a function named `my_custom_autoloader`
3. We then instantiate a class named `SomeClass1` without first having required its php file.

So what happens when this script is run?

1. First, PHP realizes that there is not yet a class named `SomeClass1` loaded, so it begins executing registered autoloaders.
2. Then, PHP will execute the autoload function we wrote (named `my_custom_autoloader`), and it will pass in the string `SomeClass1` as the value for `$class_name`.
3. Our function will define the file as `$file = __DIR__.'/includes/SomeClass1.php';`, look for its existence (`file_exists()`), and will `require_once __DIR__.'/includes/SomeClass1.php';` if found, resulting in our class’s PHP file being automatically loaded.

Huzzah! We now have a very simple autoloader that will automatically load class files as those classes are instantiated for the first time. In a decent sized project, we have just saved ourselves from writing hundreds of lines of code to include files.

## What are PHP Namespaces?

Namespaces are a way to encapsulate like-functionality or properties. A very easy (and somewhat practical) way of thinking of this is like an operating system’s directory structure:

> As a concrete example, the file foo.txt can exist in both directory /home/greg and in /home/other, but two copies of foo.txt cannot co-exist in the same directory.
> 
> In addition, to access the foo.txt file outside of the /home/greg directory, we must prepend the directory name to the file name using the directory separator to get /home/greg/foo.txt.

The way you define a namespace is at the top of a PHP file, using the `namespace` keyword:

```
<?phpnamespace Jonathan;function do_something() {    echo "this function does a thing";}
```

In the above example we have encapsulated the `do_something()` function within the `namespace` of `Jonathan`. This implies a number of things, but most importantly it means that neither of those things will conflict with the other functions of the same name in the global scope.

For example, say we have the above code in its own file name “jonathan-stuff.php”, and in a separate file we have the following code:

```
<?phprequire_once "jonathan-stuff.php";function do_something(){  echo "this function does a completely different thing"; }do_something();// this function does a completely different thing
```

See, no conflict there. We have 2 functions named `do_something()` that are able to co-exist with each other. So now all we have to do is figure out how to access the namespaced function. This is done with a syntax that is very similar to a directory structure, with backslashes:

```
<?php\Jonathan\do_something();// this function does a thing
```

The above code is saying, “execute the function named `do_something()` that resides within the Jonathan namespace”.

This also (and more commonly) is used with classes. For example:

```
?phpnamespace Jonathan;class SomeClass { }
```

Which can be instantiated like this:

```
<?php$something = new \Jonathan\SomeClass();
```

With namespaces, very large projects can contain many classes that share the same name without any conflicts. Pretty sweet huh?

### What is the point of all this? What problems do Namespaces solve?

To answer this question, we need to look back in time to a PHP without namespaces. Previous to PHP version 5.3, we could not encapsulate our classes, there was always a risk of conflicting with another class of the same name.

It was (and still is to some degree) not uncommon to prefix class names resulting in something more like this:

```
<?phpclass Jonathan_SomeClass { }
```

As you may imagine, the larger the code base, the more classes, the longer the prefixes. You shouldn’t be surprised to open an old PHP project and find a class name more than 60 characters long, like:

```
<?php class Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator { }
```

“Well, wait a second”, you say, “what’s the difference between writing that versus `\Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator`?”

That’s a great question! The answer lies in the ease of using that class more than once in a given context.

Imagine that we had to make use of a long class name multiple times within a single PHP file. Currently, we have two ways of doing this

**Without** namespaces:

```
<?phpclass Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator { }    $a = new Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator();$b = new Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator();$c = new Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator();$d = new Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator();$e = new Jonathan_SomeEntity_SomeBundle_SomeComponent_Validator();
```

Oof, that’s a lot of typing. Well, what about **with** namespaces?

```
<?phpnamespace Jonathan\SomeEntity\SomeBundle\SomeComponent;class Validator { }
```

… and elsewhere in the code:

```
<?php$a = new \Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator();$b = new \Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator();$c = new \Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator();$d = new \Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator();$e = new \Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator();
```

Well, that certainly isn’t much better. Luckily, there is a third way, leveraging the `use` keyword to pull a namespace.

### Enter, the `use` keyword

The `use` keyword “imports” a given namespace into the current context, allowing you to make use of its contents without having to refer to its full path each time.

```
<?phpnamespace Jonathan\SomeEntity\SomeBundle\SomeComponent;class Validator { }
```

… now we can do this:

```
<?phpuse Jonathan\SomeEntity\SomeBundle\SomeComponent\Validator;$a = new Validator();$b = new Validator();$c = new Validator();$d = new Validator();$e = new Validator();
```

Aside from encapsulation, **importing is the real power of namespaces**.

Now that we have an idea of what both autoloading and namespaces are, let’s combine them together to create a reliable means of organizing our project files.

## PSR-4: The Standard for PHP Autoloading & Namespaces

PHP Standard Recommendation (PSR) 4 is a commonly used pattern for organizing a PHP project so that the namespace for a class matches the relative file path to the file of that class.

For example, if you are working within a project that makes use of PSR-4 and you are dealing with a namespaced class named `\Jonathan\SomeBundle\Validator();`, you can be sure that the file for that class can be found in this relative location in the file system: `<relative root>/Jonathan/SomeBundle/Validator.php`.

Just to drive this point home, here are some more examples of where a PHP file exists for a class that is within a project making use of PSR-4:

|Namespace & Class|File Location|
|---|---|
|`\Project\Fields\Email\Validator()`|`<relative root>/Project/Fields/Email/Validator.php`|
|`\Acme\QueryBuilder\Where`|`<relative root>/Acme/QueryBuilder/Where.php`|
|`\MyFirstProject\Entity\EventEmitter`|`<project root>/MyFirstProject/Entity/EventEmitter.php`|

Note: this isn’t 100% accurate, as each component of a project has its own relative root, but don’t discount this information–

**Knowing that PSR-4 implies the file location of a class will help you easily find any class within very large projects**.

### How does PSR-4 work?

Another great question! The answer to that question is simple, it’s done with an autoloader function!

Let’s take a look at one PSR-4 example autoloader function to get an idea of how it works:

```
<?phpspl_autoload_register( 'my_psr4_autoloader' );/** * An example of a project-specific implementation. * * @param string $class The fully-qualified class name. * @return void */function my_psr4_autoloader($class) {    // replace namespace separators with directory separators in the relative     // class name, append with .php    $class_path = str_replace('\\', '/', $class);        $file =  __DIR__ . '/src/' . $class_path . '.php';    // if the file exists, require it    if (file_exists($file)) {        require $file;    }}
```

Now let’s walk through what is going on here, assuming we have just instantiated the following class: `new \Foo\Bar\Baz\Bug();`

1. PHP executes our autoloader with the `$class` parameter using a string value of `$class = "\Foo\Bar\Baz\Bug"`
2. We use `str_replace()` to change all backslashes into forward slashes (like most directory structures use), essentially turning our namespaces into a directory path.
3. We look for the existence of that file in the location `<relative root>/src/Foo/Bar/Baz/Bug.php`
4. And finally, we load that file if it is found.

TL;DR – We changed `Foo\Bar\Baz\Bug` to `/src/Foo/Bar/Baz/Bug.php` and try to find that file.

## Composer – PHP Package manager – Does autoloading for you

Composer is a command line PHP package manager. You may have seen a project before with a `composer.json` file in its root directory. This file tells Composer about your project, including your project’s dependencies.

Example of a very simple `composer.json` file:

```
{    "name": "jonathan/example",    "description": "This is an example composer.json file",    "require": {        "twig/twig": "^1.24"    }}
```

This project is named “jonathan/example” and has 1 dependency of the Twig templating engine (at version 1.24 or higher).

With composer installed on my system I can use this json file to download my project’s dependencies, and in doing so, composer will generate an `autoload.php` file that will automatically handle autoloading the classes in all your dependencies.

![Screenshot of Composer project](https://www.daggerhartlab.com/wp-content/uploads/composer-autoloading-phpstorm-300x241.png)

If I include this new file in my project, all my dependencies’ classes will be automatically be loaded as I need them in my project!

To re-iterate: **Because of the PSR-4 standard and its wide-spread adoption, Composer can generate an autoloader that will automatically handle loading your dependencies as you instantiate them within your project.**

Learn more about [Creating your own PHP Namespaces and Autoloader with Composer](http://composer-autoloader-the-easiest-autoloading-in-php/).

Additional resources on used for this post:

- PHP Documentation
    - [spl_autoload_register](http://php.net/manual/en/function.spl-autoload-register.php)
    - [PHP Namespace Keyword](http://php.net/manual/en/language.namespaces.php)
    - [Use Keyword (Importing namespaces)](http://php.net/manual/en/language.namespaces.importing.php)
- [PSR-4](http://www.php-fig.org/psr/psr-4/)
- [Composer](https://getcomposer.org/)