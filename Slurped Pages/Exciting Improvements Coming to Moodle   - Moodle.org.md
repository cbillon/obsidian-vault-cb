---
link: https://moodle.org/mod/forum/discuss.php?d=466385
excerpt: We’re making key updates to Moodle that will enhance security, streamline deployment, and modernize our codebase. These changes, which will impact how you develop and deploy Moodle, are designed to align with modern best practices and make future improvements easier.
tags:
  - moodle
  - improvements
slurped: 2025-03-10T09:00
title: "Moodle in English: 🚀 Exciting Improvements Coming to Moodle   | Moodle.org"
---

We’re making key updates to Moodle that will **enhance security, streamline deployment, and modernize our codebase**. These changes, which will impact how you develop and deploy Moodle, are designed to align with modern best practices and make future improvements easier.

Over the years, deployment processes and best practices have evolved. Some decisions made in Moodle’s early development are no longer recommended, and we’re now addressing this technical debt to ensure Moodle remains robust and adaptable.

---

#### 📌 What’s Changing?

In simple terms, we are **moving all Moodle code into a new `public` sub-directory**.

This long-requested change brings multiple benefits, including:

- **Improved security** by reducing the potential exposure of sensitive files
- **Easier plugin installation and deployment**
- **Support for Composer**, enabling more efficient dependency management
- **Encouragement of modern development practices**, such as proper autoloading and routing

##### 🗂 Files & Directories That Will Remain Outside `public`

Initially, the following files and directories will **not** be moved into `public`:

- Moodle’s `config.php` file
- Composer & NodeJS configuration files
- The `vendor` and `node_modules` directories
- [Metadata](https://moodle.org/mod/glossary/showentry.php?eid=184&displayformat=dictionary "Glossary of common terms: metadata") and configuration files for various tools

Over time, additional content will be relocated to further enhance security.

---

#### 🔧 How Will This Affect You?

To ensure a smooth transition, you’ll need to make the following updates:

##### 🗂️ Update Your Document Root

Point your server’s Document Root to the new `public` directory instead of the Moodle root. This is typically a **one or two-line configuration change** for most web servers. If you're using a managed hosting service, you may need to contact your provider.

##### 🗺️ Update Routing Engine Configuration

The Moodle Routing Engine was introduced as an _optional_ feature in Moodle 4.5, but it will become **compulsory in Moodle 5.0** ([MDL-82565](https://tracker.moodle.org/browse/MDL-82565)). In **Moodle 5.1**, it will require some reconfiguration due to the directory structure changes.

##### 🔨 Install Composer & NodeJS Dependencies

You’ll need to add the following steps to your deployment process:

```
composer install --no-dev  
npm ci  
npm run compile
```

These ensure that required dependencies are installed and compiled correctly.

---

#### 📅 When Will This Change Happen?

The new directory structure is ready but will not be applied until Moodle 5.1 (following the Moodle 5.0 release). This gives developers time to adjust their processes before the changes take effect.

---

#### 🔍 What About [Git](https://moodle.org/mod/glossary/showentry.php?eid=10110&displayformat=dictionary "Glossary of common terms: Git"), Automated Testing & Tooling?

- Git & Branch Switching: Git can handle these changes through rebasing and patching, but switching between the old and new structure may require adjustments, such as updating `.git/info/excludes`.
- Automated Testing & External Tools: If you use tools like MDK or moodle-plugin-ci, you may need to update your configurations. We’ll provide guidance closer to the transition date.

---

#### 📚 Learn More & Get Ready

We encourage you to start preparing now by reviewing these resources:

- 📌 [Roadmap proposal](https://tracker.moodle.org/browse/IDEA-75)
- 📌 [MDL-83424](https://tracker.moodle.org/browse/MDL-83424)
- 📌 [Project documentation](https://moodledev.io/general/projects/directoryrestructure)

If you have any questions or need help, join the discussion in the [Moodle Community Forums](https://moodle.org/course/view.php?id=5) or reach out to us on the [Moodle Developer Chat](https://matrix.to/#/%23moodledev:moodle.com).