---
link: https://centamori.com/index.php?slug=basics-of-web-development
excerpt: Have we returned to the basics of web development? A journey through
  Transaction Script, MVC, and SPAs to see how HTMX and Livewire are shaping the
  future.
slurped: 2026-01-02T09:16
title: Ivan Centamori
---

## Have we returned to the basics of web development?

If one observes the history of web development with detachment, one might fall into the error of seeing it as a circle closing upon itself. Or worse, as a schizophrenic series of passing fads where "everything changes so that nothing changes."

In the eyes of a senior analyst, however, the movement is not circular, but a **spiral**. We revisit concepts we thought were outdated (server-side rendering, monoliths, centralized state management), but we do so each time from a higher vantage point, with more powerful tools and a greater awareness of the trade-offs involved.

The fatigue that many developers feel today – the so-called _JavaScript Fatigue_ or the feeling of being overwhelmed by technological churn – often stems from a lack of this prospective vision. It is not about "going back" because new tools are difficult, but understanding that every architecture was a necessary response to the limits of the previous one, bringing with it new problems that the subsequent iteration sought to solve.

In this article, we will retrace the three great eras of web architecture, not out of nostalgia, but to lucidly analyze the _trade-offs_ of each phase. Only in this way can we understand why technologies like HTMX, React Server Components, or Laravel Livewire are not a "return to the past," but the necessary dialectical synthesis for the future.

---

## 1. Transaction Script

At the dawn of the dynamic web, the dominant architecture was what Martin Fowler defines as **Transaction Script**. In this paradigm, there were no stratified abstractions: each file on the server represented a complete logical unit responsible for handling a specific transaction from start to finish, coordinating input, business logic, and output in a single procedural flow.

The mental model was extremely simple: an HTTP request maps 1:1 to a file on the server's disk. Does a `GET /login.php` arrive? The server executes `login.php`. Does a `POST /save_user.php` arrive? The server executes `save_user.php`.

There were no routers, no middleware, no containers for Dependency Injection. There was only the code.

### Example of a script

Here is an example of what we would call "spaghetti code" today, but which at the time was the industry standard for moving billions of dollars in e-commerce:

```
<?php
// user_manager.php

// 1. DB Connection (often copied in every file or included via require)
$conn = mysql_connect("localhost", "root", "");
mysql_select_db("app_db");

// 2. Write Logic Handling (POST)
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'];
    $email = $_POST['email'];

    // Immediate validation, "inline"
    if (strlen($username) < 3) {
        $error = "Username too short";
    } else {
        // No ORM, direct SQL
        // Note: at the time security (escaped string) was often an afterthought
        $sql = "INSERT INTO users (username, email) VALUES ('$username', '$email')";
        mysql_query($sql);

        // Post-submission redirect
        header("Location: user_manager.php?success=1");
        exit;
    }
}

// 3. Read Logic Handling (GET)
$users = mysql_query("SELECT * FROM users ORDER BY id DESC LIMIT 10");
?>

<!-- 4. Rendering (View) mixed with logic -->
<!DOCTYPE html>
<html>
<head><title>User Management</title></head>
<body>
    <?php if (isset($_GET['success'])): ?>
        <div style="background:green; color:white">User saved!</div>
    <?php endif; ?>

    <?php if (isset($error)): ?>
        <div style="background:red; color:white"><?= $error ?></div>
    <?php endif; ?>

    <h1>User List</h1>
    <ul>
        <?php while ($row = mysql_fetch_assoc($users)): ?>
            <li>
                <strong><?= $row['username'] ?></strong> 
                (<?= $row['email'] ?>)
            </li>
        <?php endwhile; ?>
    </ul>

    <h2>Add User</h2>
    <form method="post">
        <input type="text" name="username" placeholder="Username">
        <input type="email" name="email" placeholder="Email">
        <button type="submit">Save</button>
    </form>
</body>
</html>
```

### Architectural analysis

Today we look at this code with horror due to the lack of separation, security fragility, and difficulty in testing. But let's stop for a moment to look at the objective advantages, the ones we lost along the way.

- **Locality of Behavior (LoB)**: If you need to understand how the "User Management" page works, you only need to open **one file**. The validation is there. The query is there. The HTML is there. The "cognitive load" to understand a single feature is extremely low. All context is right before your eyes (or at least, in the same vertical scroll, even if line numbers could sometimes reach 5 digits).
- **Latency and Simplicity**: There is no framework that needs to boot, load 50 service providers, resolve dependency reflection, grind through middleware. PHP starts, executes, dies. The _Stateless_ and _Shared Nothing_ architecture allowed for trivial horizontal scalability.

### Giants standing on scripts

It is fundamental to dispel a myth: this architecture was not "beginner stuff." On the contrary, it was the launch engine for today's biggest web giants, systems that handled unimaginable traffic.

**Facebook** is the archetypal example. For years, the world's largest social network was essentially a gigantic collection of PHP scripts. Zuckerberg's _"Move Fast and Break Things"_ philosophy was made possible precisely by _Locality of Behavior_. An engineer could open a single file, modify the logic, save, and see the result. There were no 20-minute build pipelines or impenetrable abstraction layers. The "ugliness" of the code was the price to pay for an iteration speed that competitors (often stuck in over-engineered Java/J2EE stacks) could not match.

**WordPress** too, which powers over 40% of the web, demonstrated that simplicity and accessibility can lead to success. Its architecture based on global hooks and procedural functions horrifies Object-Oriented Programming purists, but it democratized online publishing more than any other technology. Its resilience and the ease with which an average user can paste a snippet into `functions.php` to modify site behavior are proof that accessibility and simplicity often beat academic purity.

### The collapse of the model

Why did we abandon this apparent paradise? Because it didn't scale with **domain complexity**. When business rules become hundreds and data interactions deepen, _Locality of Behavior_ turns into a maintenance nightmare. The problem was not the technology itself, but the inability to manage growth without a formal structure. The code became "spaghetti code" not out of ill will, but out of necessity: every new feature required touching huge files where business logic, database queries, and HTML markup were fused in an unbreakable bond.

- **Duplicate Logic:** If you needed to change email validation logic, you had to search for it with `CTRL+F` in 50 different files.
- **Untestable:** How do you unit test business logic if it's glued to `$_POST` and `mysql_query`? You can't.
- **Security Nightmares:** Forget a `mysql_real_escape_string` in a single file and you are breached.

We needed order. We needed engineering.

---

## 2. Model View Controller (MVC)

Around 2005-2010, the PHP world (and Ruby, and Python) began to professionalize. "Full-Stack Frameworks" were born: **Symfony**, **Zend Framework**, **Ruby on Rails**, **Django**.

The answer to the chaos of procedural scripts was the systematic adoption of the **Model-View-Controller (MVC)** pattern. As web applications became more ambitious, the need for a formal structure became imperative. It was no longer just about making a page work, but about making code maintainable by teams composed of dozens of developers. MVC introduced the concept of "Separation of Concerns," dividing the application into three distinct logical components, each with a well-defined scope of action.

The **Model** represents the beating heart of the application. It is not simply a representation of the database table, but the place where business "truth" resides. In this phase, we saw the rise of **ORMs (Object-Relational Mapping)** like Hibernate in Java, Doctrine in PHP, or ActiveRecord in Ruby on Rails. The Model handles data validation, complex calculations, and persistence, isolating the rest of the system from the complexity of SQL queries and the physical structure of the database.

The **View** is the layer dedicated exclusively to presentation. With the advent of MVC frameworks, HTML stopped being generated via messy string concatenation, giving way to **Template Engines** (like Blade, Twig, ....). These tools allowed writing clean markup, enriched by a declarative syntax for loops and conditions, ensuring that visualization logic did not interfere with application logic. The View receives "passive" data and limits itself to transforming it into the final visual representation for the user.

Finally, the **Controller** acts as the conductor and entry point for every interaction. Its job is to intercept the HTTP request, interpret the user's intentions, and invoke the necessary actions on the Models. Once results are obtained, the Controller decides which View to render and passes the necessary data to it. This separation ensures that the navigation flow is distinct from both the data structure and its representation, allowing individual components to evolve independently.

### The cost of abstraction

The code became clean, testable, reusable. We introduced concepts from Enterprise Java into the world of web scripting.

1. **Route (`routes/web.php`)**: Defines the URL.
    
    ```
    Route::post('/users', [UserController::class, 'store']);
    ```
    
2. **Controller (`UserController.php`)**: Coordinates request and response.
    
    ```
    public function store(StoreUserRequest $request, CreateUserAction $action) {
       $action->execute($request->validated());
       return back();
    }
    ```
    
3. **Form Request (`StoreUserRequest.php`)**: Isolates validation logic.
    
    ```
    public function rules() {
       return [
           'username' => 'required|min:3',
           'email' => 'required|email'
       ];
    }
    ```
    
4. **Service/Action (`CreateUserAction.php`)**: Contains pure business logic.
    
    ```
    public function execute(array $data) {
       return User::create($data);
    }
    ```
    
5. **Model (`User.php`)**: Represents the entity and persistence.
    
    ```
    class User extends Model {
       protected $fillable = ['username', 'email'];
    }
    ```
    
6. **Migration (`create_users_table.php`)**: Defines database schema.
    
    ```
    Schema::create('users', function ($table) {
       $table->id();
       $table->string('username');
       $table->string('email');
    });
    ```
    
7. **View (`index.blade.php`)**: The UI template.
    
    ```
    <form action="/users" method="POST">
       <input type="text" name="username">
       <input type="email" name="email">
       <button>Save</button>
    </form>
    ```
    

We destroyed _Locality of Behavior_. To understand "what happens when I click Save," a junior developer has to jump through 7 files and understand the framework's lifecycle. The spatial complexity of the code exploded.

### The UX problem: The "round trip"

While we engineers enjoyed the cleanliness of backend code, users started to suffer. In a classic Server-Side MVC architecture (SSR), every interaction requires a **full page load**.

Does the user click "Add to Cart"?

1. Browser sends POST.
2. Server processes.
3. Server renders the ENTIRE HTML page.
4. Browser downloads everything, flashes white, and redraws.

With the explosion of the smartphone market, the way we consume digital content changed radically. Native applications, built specifically for iOS and Android, introduced an extremely high standard of interaction: fluid transitions, immediate feedback, and seamless navigation that redefined every user's expectations.

In this new scenario, the classic "round trip" of the traditional web began to show all its limits. Seeing the screen turn white for a fraction of a second at every click, or waiting for the entire page to reload for a small change, became an experience perceived as slow and "clunky." The contrast between the responsiveness of installed apps and the clunkiness of the browser had become unacceptable.

---

## 3. Single Page Applications (SPA)

The industry's response, between 2012 and 2020, was drastic: **abandon HTML server-side rendering** in favor of logic entirely managed by the client. The browser stops being a simple document viewer and becomes the execution environment for a full-fledged application that consumes raw data. This paradigm shift bridged the performance gap with native apps, offering reactive interfaces and instant navigation.

The era of **SPAs** (React, Angular, Vue) was born. The architecture breaks into two distinct worlds connected by a thin thread:

1. **Frontend (The Client):** A complete JavaScript application running in the browser. Handles routing, validation, rendering, state.
2. **Backend (The API):** A "mute" server that speaks only JSON. Stateless, RESTful (or GraphQL).

### The explosion of accidental complexity

At first, it seemed great. Total separation! Frontend and Backend developers could work in parallel. But soon, we realized we had created a monster of complexity.

#### 1. The multiplication of states

Before, state was in the Database. Period. With SPAs, we have state in the DB, state in transit in the API response, state in the client's Redux/Pinia Store, and state in the local DOM. Synchronizing these states became **the hardest problem** of modern frontend development. (Cache invalidation anyone?)

#### 2. The limits of JSON

The backend sends raw data (`{"id": 1, "username": "Mario"}`) to the client, which must _know_ how to render it. This creates **High Coupling**, meaning if you change a field name in the DB, you have to update:

- The Backend Model
- The API Resource
- The TypeScript Interface in Frontend
- The React Component

#### 3. The bundle size and hydration problem

We arrived at the paradox where, to display a simple list of users, the browser has to download and process hundreds of kilobytes of framework and application logic. This initial "payload" not only slows down loading time but forces the client into enormous computational work before it can even show real content. It is the inefficiency of distribution: we often ship the entire factory (the framework) to the customer, instead of simply delivering the finished product (the HTML). The browser must:

1. Download empty HTML (`<div id="app"></div>`).
2. Download JS.
3. Parse and execute JS.
4. Make API calls for data.
5. Render the DOM.

The result? Loading spinners everywhere. To mitigate this, we invented _Server Side Rendering (SSR) for SPAs_ (Next.js, Nuxt), adding another layer of monstrous complexity (Hydration, Re-Hydration, double rendering server/client).

We had recreated the problems of point 1 (spaghetti code, but in JS) with the distributed complexity of point 3.

---

## 4. HTML-over-the-wire

And here we are today. The spiral has completed a turn and expert developers, tired of managing distributed state management and fragile build pipelines, started asking a question:

> "What if the server sent the HTML, but not the whole page, just the little piece needed?"

This is the philosophy behind **HTMX**, **Hotwire (Rails)**, **Laravel Livewire**, and, in a different but parallel way, **React Server Components (RSC)**.

### The paradigm shift

The idea is simple but revolutionary compared to the SPA decade: **The server is the Single Source of Truth**.

Instead of: `Click -> JS intercepts -> Fetch JSON -> JS updates State -> JS recalculates VDOM -> DOM Update`

We have: `Click -> HTML Attribute intercepts -> Fetch partial HTML -> Swap in DOM`

### HTMX

Let's imagine the same "User Save" functionality from point 1, but modernized using HTMX.

```
<!-- index.html -->
<!-- The list refreshes automatically listening for the 'newUser' event -->
<ul id="user-list" hx-get="/users/fragment" hx-trigger="newUser from:body">
    @include('partials.user-list')
</ul>

<!-- The form sends data via AJAX and, if successful, emits a signal -->
<form hx-post="/users" hx-swap="none">
    <input type="text" name="username">
    <button type="submit">Save</button>
</form>
```

```
// UserController.php
public function store(Request $request) {
    // 1. Validation and DB (all the power of Laravel/Symfony)
    $user = User::create($request->validate([...]));

    // 2. Instead of JSON, or redirect, we send a special header
    // saying to the frontend: "Hey, something happened!"
    return response()->noContent()->withHeaders([
        'HX-Trigger' => 'newUser'
    ]);
}
```

### Why is this the winning synthesis?

This approach allows recovering the advantages of traditional server-side development without giving up the fluidity and reactivity typical of modern Single Page Applications. Here are the main reasons why this architecture represents an extremely effective synthesis:

1. **Locality of Behavior Regained:** Looking at the HTML, I understand exactly what the button does (`hx-post="/users"`). I don't have to look for the `handleSubmit` function in a separate JS file.
2. **Zero Client-Side State Management:** There is no store. The state is in the database. The HTML is always a faithful reflection of the server state.
3. **Reduced Payload:** I don't download 2MB of React/Angular. I download only small HTML fragments when needed.
4. **"SPA-Like" UX:** The user sees no refresh. They perceive fluidity.

---

## 5. Choosing the right complexity

There is no "perfect architecture." There is only the architecture suitable for the problem you are solving.

The lesson of these 20 years is that **Place-Oriented Architecture** (as HTMX creators call it) or **Locality of Behavior** are values we sacrificed too quickly on the altar of separation at all costs.

As a Tech Lead, your job is not to follow the hype of micro-frontends or "Edge-first" architectures. Your job is to evaluate the **Complexity Budget**.

- Are you building a video editing tool in the browser? Or Google Maps? **Use an SPA.** You need complex state management on the client.
- Are you building a management system, an e-commerce site, a B2B dashboard, a blog, a social network? (That is, 90% of the web). **Seriously consider HTML-over-the-wire.**

Recovering simplicity is not a step backward. It is the definitive sign of maturity. It means understanding that the best code is the code you don't have to write (or maintain).

We have returned to the basics, but with superpowers.