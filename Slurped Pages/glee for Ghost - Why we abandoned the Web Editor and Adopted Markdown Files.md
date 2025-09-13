---
link: https://journal.hexmos.com/glee-2/
byline: Lince Mathew
site: Hexmos Journal
date: 2023-09-03T16:13
excerpt: Discussing the challenges we faced with the Web Editor on the Ghost blogging platform that led us to abandon its use and create a CLI tool called 'glee.
twitter: https://twitter.com/@hexmostech
slurped: 2025-08-31T20:23
title: "glee for Ghost: Why we abandoned the Web Editor and Adopted Markdown Files"
tags:
  - ghost
  - cli
  - markdown
  - glee
---

- [Content writing problem](https://journal.hexmos.com/glee-2/#content-writing-problem)
- [glee: A CLI Idea](https://journal.hexmos.com/glee-2/#glee-a-cli-idea)
- [Evaluating Our Options: Markdown files vs the Ghost Editor](https://journal.hexmos.com/glee-2/#evaluating-our-options-markdown-files-vs-the-ghost-editor)
- [From Writing to Publishing](https://journal.hexmos.com/glee-2/#from-writing-to-publishing)
    - [Writing the Markdown File](https://journal.hexmos.com/glee-2/#writing-the-markdown-file)
        - [Metadata](https://journal.hexmos.com/glee-2/#metadata)
        - [Markdown Content](https://journal.hexmos.com/glee-2/#markdown-content)
            - [Markdown to HTML](https://journal.hexmos.com/glee-2/#markdown-to-html)
    - [Publishing on the Ghost Platform](https://journal.hexmos.com/glee-2/#publishing-on-the-ghost-platform)
        - [Configuring Ghost API /S3](https://journal.hexmos.com/glee-2/#configuring-ghost-api-s3)
        - [JWT TOKEN](https://journal.hexmos.com/glee-2/#jwt-token)
            - [AWS-S3 Configuration](https://journal.hexmos.com/glee-2/#aws-s3-configuration)
        - [Ghost Admin API](https://journal.hexmos.com/glee-2/#ghost-admin-api)
            - [Creation using Post API](https://journal.hexmos.com/glee-2/#creation-using-post-api)
            - [Updating blog posts using PUT API](https://journal.hexmos.com/glee-2/#updating-blog-posts-using-put-api)
    - [glee command](https://journal.hexmos.com/glee-2/#glee-command)
- [Team Collaboration](https://journal.hexmos.com/glee-2/#team-collaboration)
- [Conclusion](https://journal.hexmos.com/glee-2/#conclusion)
    - [Future works](https://journal.hexmos.com/glee-2/#future-works)

## Content writing problem

At [Hexmos](https://journal.hexmos.com/glee-2/hexmos.com), we strive to share our learnings on a weekly cadence through articles, videos, and mini tools throughout the year.

For article delivery, we have a self-hosted [Ghost](https://ghost.org/?ref=journal.hexmos.com) Platform known as [Hexmos Journal](https://journal.hexmos.com/). Ghost Blog Platform has numerous features for creating excellent content but we face some difficulties continuously, such as:

1. Suboptimal editor for Engineers: Writing articles directly in the Ghost default editor sometimes strains us too much, especially while using Markdown syntax and adjusting images in the content. It takes too much time for us to align the content properly within the editor.
2. Content history control: Another problem is content history control. If we want to revert the new paragraph to an older one or cross-check the difference, it's not possible in the Ghost editor because Ghost is overwriting on updating the content and it lacks version management.
3. Team collaboration: Two team members can't simultaneously work together in the Ghost editor. Neither member's changes will be saved in the Ghost editor, leading to data loss.

## `glee`: A CLI Idea

To tackle the issue mentioned earlier, we came up with the idea of building a tool called [`glee`](https://github.com/HexmosTech/glee?ref=journal.hexmos.com) on top of the Ghost Blog Platform. `glee` supports version management and collaboration using git. Also directly publishes content in Markdown format onto Hexmos Journal, our Ghost Blog Platform, without interfering with the Ghost editor.

Now the question is, how to build the tool? Ghost provides Admin APIs to create and manage the content in Ghost Platform. This means that everything Ghost Admin can do is also possible with the API.

So basically we need to do the following - We can write the content in Markdown syntax using VS Code or any other editors - Convert the markdown into HTML format. - Publish it into Ghost directly using Ghost Admin API.

Also, our tool needs to support Markdown extensions such as

- Code formatting
- Table of contents
- Table, etc

For managing the ghost blog, we need a mechanism to manage the metadata such as - Author Details - Images - Slug, etc

## Evaluating Our Options: Markdown files vs the Ghost Editor

When evaluating the options between Markdown files and the Ghost Editor, it's important to consider the perspectives of key individuals involved in Ghost, such as **John O'Nolan**, **Hannah Wolfe**, and **Peter Schulz**.

Supporters of the Ghost Editor philosophy argue that:

- **Flexibility with Dynamic Editor Blocks and Widgets**: The Ghost Editor offers dynamic editor blocks and widgets, providing flexibility that plain text alone cannot match. This allows users to easily incorporate complex styling, multimedia elements, and interactive components into their content, enhancing the overall user experience.
    
- **Ease of Use for Non-Technical Users**: The user-friendly interface of the Ghost Editor makes it accessible to non-technical users who may not be familiar with Markdown syntax. This can empower a broader range of content creators to express their ideas without the need for technical expertise.
    

On the other hand, we argue that:

- **Simplicity and Performance**: Markdown files align with a philosophy of simplicity and lightweight tools. By using plain text files, users can avoid unnecessary complexity from using an editor.
    
- **Version Control and Collaboration**: Markdown files excel in version control and collaboration. With Markdown files, it is easier to track changes, compare revisions, and collaborate with multiple authors. This aligns with core values of transparency, teamwork, and efficient content management.
    
- **Text-Focused Content**: For content that primarily focuses on text with minimal styling needs, Markdown syntax provides a straightforward and efficient way to format text. This simplicity can be advantageous for authors who prioritize the clarity and readability of their content.
    
- **Automation and Tooling**: Markdown lends itself well to automation and tooling. Various tools and scripts can be leveraged to automate processes such as publishing, formatting, and generating content from Markdown files. This enhances productivity and efficiency in content management workflows.
    
- **Long-Term Sustainability**: Markdown files offers more future-proofing compared to proprietary editor software. As an open standard widely adopted across platforms and tools, Markdown ensures that content remains accessible and editable in the long run, even if specific editor software becomes obsolete.
    

While the Ghost Editor philosophy provides advantages in terms of intuitive visual content creation, flexibility, and ease of use, the Markdown file approach prioritizes simplicity, version control, collaboration, automation, and long-term sustainability, hence we chose to abandon the ghost editor.

## From Writing to Publishing

In `glee`, we can split mainly into two stages such as:

1. Writing the Markdown file
2. Publishing in the Ghost Blog Platform

### Writing the Markdown File

The Markdown file used by `glee` consists of two sections:

1. Metadata
2. Content

Let me explain each in detail using this example `sample.md` file

---
title:  'testing sample Markdown file'
authors:
- example@gmail.com
tags: []
featured: false
status: draft
excerpt: null,
feature_image: ./smiley.png
slug: testing-glee
---

# My Simple Markdown File

This is a basic Markdown file with some common formatting elements.

## Headers

You can create headers using the `#` symbol. There are six levels of headers:

#### Header 4
##### Header 5
###### Header 6

## Text Formatting

You can make text **bold** using double asterisks or double underscores, and you can make it *italic* using single asterisks or single underscores.

The first section which is in yaml format is the metadata. We use metadata to define the title, authors of the blog post, slug, status of the post etc.

What follows, is in Markdown format. In the content section, you can add a Table of content, images, and table and it also support code formatting.

#### Metadata

Let's discuss the metadata section of the Markdown file in depth. Consider this `YAML` preface in the sample.md file

---
title:  'testing sample Markdown file'
authors:
- example@gmail.com
- example2@gmail.com
tags: []
featured: false
status: draft
excerpt: null,
feature_image: ./smiley.png
slug: testing-glee
---

- `title` : The main title of the blog post. Possible to update later using the `glee` tool
- `authors`: The `authors` field in the Markdown frontmatter can specify multiple staff emails that is, the email of the author.
- `tags`: specify the post tag in the tag field.
- `featured`: If `featured` is `true` the post has a higher preference in your blog.
- `status`: It determines the status of the post. Pick status: draft or status: published as required.
- `excerpt`: Creating excerpts of posts
- `feature_image`: Include the path for the feature image of the blog.
- `slug`: If the post doesn't contain a `slug` field (post name in the URL), then `glee` will not publish. This is to help with future updates/edits from the Markdown file.

#### Markdown Content

The content of the Markdown is defined below the YAML preface. It supports standard markdown plus many useful extensions

###### Markdown to HTML

We use the Python library [`Markdown`](https://pypi.org/project/Markdown/?ref=journal.hexmos.com) for converting Markdown-formatted text into HTML.

**Handling Extensions**

Currently, we include five extensions in `glee` for better markdown support such as:

1. TocExtension(): Allows to generate a Table of Contents (TOC) from headings in the Markdown file.
    
2. FencedCodeExtension(): Allows to create code blocks within Markdown documents using fenced code blocks.
    
3. CodeHiliteExtension(): Allows to highlight code syntax within Markdown documents.
    
4. ImgExtExtension(): Allows support for extra features related to images in Markdown documents.
    
5. TableExtension(): Allows to create tables in Markdown documents.
    

Finally `glee` posts the converted HTML into Ghost Blog Platform using Ghost Admin APIs.

### Publishing on the Ghost Platform

#### Configuring Ghost API /S3

Configuration is handled using a hidden file called .glee.toml located in your home directory

# refer https://github.com/HexmosTech/glee#configuration for configuration details

[ghost-configuration]
ADMIN_API_KEY = "" 
GHOST_VERSION = "v5"  # eg: v5,v4...
GHOST_URL =""

[aws-s3-configuration]
ACCESS_KEY_ID = ""
SECRET_ACCESS_KEY = ""
BUCKET = ""
S3_BASE_URL = ""

- `ADMIN_API_KEY` : Admin API keys are special codes that are used to create temporary and unique access tokens called JSON Web Tokens (JWTs). These tokens are used to verify and authorize requests made to the Ghost Admin API, such as GET, POST, and PUT requests, You can learn more about setting up `ADMIN_API_KEY` [here](https://github.com/HexmosTech/glee?ref=journal.hexmos.com#ghost-admin-api-key)
- `GHOST_VERSION` : Specifies the Ghost platform version
- `GHOST_URL` : The GHOST_URL represents the domain where your Ghost blog is hosted.
    
- `ACCESS_KEY_ID` : AWS Access key
    
- `SECRET_ACCESS_KEY` : AWS Secret access key
- `BUCKET` - S3 Bucket name
- `S3_BASE_URL` - Link to the amazon s3

#### JWT TOKEN

We need jwt token to authenticate for admin API requests. To do this, we can use glee.py’s `get_jwt()` function to generate a jwt token for doing admin API requests.

##### AWS-S3 Configuration

Presently all images in the input post are uploaded to an S3 bucket. We calculate the hash for each image and use that as the filename in s3. This ensures that each unique image is stored only once in the server and that there are no naming conflicts.

#### Ghost Admin API

Ghost provides APIs to create, update and delete posts, we will be going through how creation and updation are done via APIs

##### Creation using Post API

We use this API endpoint in `glee` to create a post.

`POST /admin/posts/?source=html`

##### Updating blog posts using PUT API

We use this API endpoint in `glee` to update a post.

`PUT /admin/posts/{id}/?source=html`

In `glee` we set the source as HTML, so that is achieved via putting `?source=html`

You can learn more about these APIs [here](https://ghost.org/docs/admin-api/?ref=journal.hexmos.com#posts)

### `glee` command

The `glee` command will read metadata from the YAML preface of your Markdown post (sample_post.md), convert the post content into HTML, store the content images in AWS S3, and then publish it to your Ghost platform.

This is how you can use this command

`glee your-post.md`

`your-post.md` is the markdown file that you would like to publish as a ghost blog

## Team Collaboration

With `glee`, collaboration within your teams becomes a breeze, thanks to the integration of Git. Here's how it works:

- _Organized Git repository_: You can create a dedicated repository for your ghost blog posts. You can also create directories for individual blog posts.
    
- _Markdown Magic_: Inside these directories, you can add a markdown file for each blog post you intend to write.
    
- _Effortless Updates_: Team members can effortlessly push their changes and pull updates from others. This ensures a smooth and collaborative environment.
    
- _Instant Publishing_: After pulling the latest changes, simply use the command `glee your_blog_name.md` to publish your blog.
    

## Conclusion

In this post, we discussed the challenges we faced with the Web Editor on the Ghost blogging platform that led us to abandon its use. These obstacles prompted our transition to Markdown files for content management, leading to us devising a solution for it in the form of a tool.

`glee` has the potential to solve real pain points that users face with content management on Ghost today. With further development, it could become a handy extension that enhances the content workflow for Ghost blogs.

### Future works

- Direct Code Injection: Code Injection provides an interface for conveniently adding analytics, styles, custom fonts, meta tags, and scripts to a Ghost site. Currently, users do not have the option to inject code directly from the Markdown.
    
- Improving the `glee` command: Improve argument handling for better command-line usability, After creating or updating a post, provide a preview link to the blog post in the terminal for easier access and review.
    
- VS Code extension: VS Code extension with features like direct, real-time preview for Ghost blog posts and Markdown reformatting.
    
- Powerful Library: More advanced and powerful Markdown library for `HTML` conversion.
    

**Github Repository**

[glee: Publish Markdown Files to Ghost Blog](https://github.com/HexmosTech/glee?ref=journal.hexmos.com)