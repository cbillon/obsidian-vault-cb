---
link: https://wendelladriel.com/blog/strategic-enums-in-php
site: Wendell Adriel
excerpt: Enumerations, commonly known as Enums are a powerful tool in Software Development and can play an important role in the Architecture and Design of applications if used correctly. In this article let's review how we can strategically use them to improve our applications.
twitter: https://twitter.com/wendell_adriel
tags:
  - php
  - enums
slurped: 2025-09-13T19:43
title: Strategic Enums in PHP
---

[

## Introduction


**Enumerations**, commonly known as **Enums** are a powerful tool in **Software Development** and can play an important role in the **Architecture** and **Design** of applications if used correctly. For a long time, we didn't have the support for this data type in PHP, but starting with PHP 8.1 we finally got them added, bringing a lot of possibilites with them. In this article let's review how we can strategically use them to improve our applications.

[

## Why to use Enums


Basically an **Enum** is a data type that works as a collection of predefined constants. For example, if we're building a **ToDo** application, we can use an **Enum** to represent the priority of a task.

```
enum Priority
{
    case LOW;
    case NORMAL;
    case HIGH;
}
```

Using an **Enum** to represent the priority of a task already gives us an important improvement for our design, instead of using a string to represent it, we can use it as a type in our `Task` entity, so when dealing with this entity in our codebase we can easily identify what are the possible values for the priority property instead of trying to guess or checking the database for it.

```
final class Task
{
    public function __construct(
        public string $title,
        public string $description,
        public Priority $priority,
    ) {
    }
}

$task = new Task(
    title: 'My First Task',
    description: 'Do something',
    priority: Priority::HIGH,
);
```

But the code above has an issue when dealing with the database, since the **Enum** is represented by an object, we can't easily use it to save the data in the database, but we have another type of **Enums** to save the day.

[

## Backed Enums

As said above, **Enums** are represented by objects, but PHP supports also **Backed Enums**, that can have a value assigned to them. Internally they are still represented by objects, but since they are represented by a value, we can easily use this value to serialize the Enum for saving it to the database for example. Let's improve our `Priority` enum and turn it into a **Backed Enum**.

```
enum Priority: string
{
    case LOW = 'low';
    case NORMAL = 'normal';
    case HIGH = 'high';
}
```

The instantiation of our `Task` entity could remain the same as before and it would still work, but we can also use it in a different way. Imagine that we receive from the UI the priority as a string value and when instantiating the `Task` entity we need to assign the Enum. We can use this approach:

```
$task = new Task(
    title: 'My First Task',
    description: 'Do something',
    priority: Priority::from($priority),
);
```

Keep in mind that we can use only `string` or `int` types for **Backed Enums**.

[

## Enum Methods

Just like normal classes, we can also define methods in our Enums and this brings a lot of possibilities when we combine the use of the `match` operator with these methods. Imagine that now we want to give a specific color for each of the priorities in our **ToDo** application. We can easily accomplish that by creating a method in our `Priority` enum.

```
enum Priority: string
{
    case LOW = 'low';
    case NORMAL = 'normal';
    case HIGH = 'high';

    public function color(): string
		{
        return match ($this) {
            self::LOW => 'green',
            self::NORMAL => 'blue',
            self::HIGH => 'red',
        };
    }
}

$task = new Task(
    title: 'My First Task',
    description: 'Do something',
    priority: Priority::HIGH,
);

$task->priority->color(); // 'red'
```

[

## Pushing Enum Methods Further

We saw above that we can define methods in our Enums and that opens a lot of possibilities in our applications. The example we saw was only to return a specific color for a priority, but we can push that boundary even further with the methods.

Imagine that we need to send a notification to the user for each task, but the type of notification is different for each priority. We have a lot of different ways of handling this situation and one of the solutions could be using our `Priority` Enum to get the correct type of notification sender that we need. First thing we need is to define a contract for the notification sender.

```
interface NotificationSender
{
    public function send(Task $task): void;
}
```

Now that we have the contract defined, we need to implement each type of sender that we need, for the simplicity of this example, I'm not going to provide the implementation for these, just list the different implementations we could have.

```
final class EmailSender implements NotificationSender
{
    public function send(Task $task): void
		{
        // Send email logic here
    }
}

final class SlackSender implements NotificationSender
{
    public function send(Task $task): void
		{
        // Send slack notification logic here
    }
}

final class PhoneMessageSender implements NotificationSender
{
    public function send(Task $task): void
		{
        // Send SMS logic here
    }
}
```

Now with our notification senders implemented, we can implement a simple method in our `Priority` Enum that would retrieve the correct sender for each priority type.

```
enum Priority: string
{
    case LOW = 'low';
    case NORMAL = 'normal';
    case HIGH = 'high';
	
	  public function notificationSender(): NotificationSender
		{
        return match ($this) {
            self::LOW => new EmailSender(),
            self::NORMAL => new SlackSender(),
            self::HIGH => new PhoneMessageSender(),
        };
    }
}
```

With this simple method, when creating a new task, we can easily send the notification using the approach below.

```
$task = new Task(
    title: 'My First Task',
    description: 'Do something',
    priority: Priority::HIGH,
);

$task->priority->notificationSender()->send($task);
```

[

## Conclusion

As you can see, **Enums** provides to us an infinite number of possibilites to improve our applications and if we use them wisely, we can design our codebase to be much cleaner and easier to maintain. My goal with this article was to show that we can go beyond the simple helper functions like getting a color or a label with our **Enums**, but also to use them to solve more complex issues and having them as a central part of the **Architecture** and **Design** of our applications.
