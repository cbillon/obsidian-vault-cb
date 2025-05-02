---
link: https://technologyconversations.com/2020/08/31/what-is-gitops-and-why-do-we-want-it/#more-4615
byline: by Viktor Farcic
site: Technology Conversations
date: 2020-08-31T10:44
excerpt: GitOps is nothing new. Or, to be more precise, the principles of GitOps
  existed long before the term was invented. But hey, that’s the pattern in our
  industry. It is the fate of all good prac…
twitter: https://twitter.com/@vfarcic
slurped: 2024-12-07T09:09
title: What Is GitOps And Why Do We Want It?
---

GitOps is nothing new. Or, to be more precise, **the principles of GitOps existed long before the term was invented**. But hey, that’s the pattern in our industry. It is the fate of all good practices to be misunderstood, so we need to come up with new names to get people back on track. That is not to say that we are in a constant loop. Instead, I tend to think of it as a periodic reset trying to eliminate misinterpretations. GitOps is one of those resets. It fosters the practices and the ideas that existed for a while now and builds on top of them.

  

GitOps assumes that a version control system (VCS) is the only source of truth. But, since all version controls worth talking about are based on Git, the name is GitOps and not VCSOps. Therefore, I’ll start using Git instead of VCS throughout the rest of this expose even though, theoretically, you could apply GitOps principles with, let’s say, SVN or Perforce. You would need to put extra effort, but it is doable.

What does it mean that Git is the only source of truth? To many, that sounds like a question with an obvious answer. Yet, many are not really applying that seemingly simple logic. So, let’s start with a simple question. Why would Git be the only source of truth?

Everything we do today is **defined as code**, at least among software engineers. Now, before you start wondering what a software engineer is, let me give a simple definition. If your work is related to the development of software, you are a software engineer. It does not matter whether you write code of an application, if you are testing, if you are deploying apps, working on infrastructure, or operating a system. You are a software engineer and, as such, you write code.

What you are not doing is clicking buttons. You do not execute all the tests manually by clicking links and filling in fields on a web page. You are not managing infrastructure through some UI. If you are doing stuff without writing code, you are in the wrong industry. You should write code. Actually, that is not correct. You **MUST be writing code**, or consider a different profession. You are hopefully young and, if code scares you, there is still time to change your calling. You can become a lawyer, a real estate agent, or whatever makes you more comfortable. If you choose to stay in the software industry, you are choosing to write code.

The only acceptable usage of UIs is for informational purposes. They are useful when observing the state of something. They are great for correlating metrics through dashboards. They are helpful when searching through logs to find the cause of an issue. But, when it comes to defining something, we write code instead of clicking buttons.

Code can be many different things, and people differ in their interpretation of what is and what isn’t code. So, let me simplify that for you. Code is something that can be interpreted by machines. If you write applications in Java, Go, Python, or any other programming language, you are writing code. That much is clear, isn’t it? If you are writing executable tests, they are code as well. If you are writing build scripts, they are also code. Heck, even if all you do is write YAML files, that’s also code. If it can be interpreted by machines, it is code.

Now that we established that everyone, you included, either writes code or is planning to change the profession, the logical outcome is that all that is stored in a version control system and that the only one that makes sense today is Git. If everything we do related to machines is defined as code, and all the code is stored in Git, it stands to reason that **Git is the only source of truth**.

The code of our applications is in Git, the tests are in Git, the configurations are in Git, and everything else is in Git. Even the documentation is today stored in Git so that machines can convert it into HTML, PDFs, or whichever other formats our users expect.

That means that, among other things, the full definition of any aspect of our systems is in Git. So, how do we operate the systems? By making changes to Git. Hence, the expression GitOps means that the operations are performed through Git.

So, the two fundamental principles of GitOps are:

- Everything is described as code
- Git is the only source of truth

Now that we narrowed down the whole GitOps into only two principles, let’s discuss the outcomes of applying them.

## The Outcomes Of The GitOps Principles

What are the outcomes of GitOps principles?

If everything is defined as code and Git is the only source of truth, the definitions in Git are describing the desired state. We push to Git what we want to have. We could, just as well, make those desires the reality by applying the desired state and, thus, converting it into the actual state manually. But that is unreliable and unproductive.

We, humans, are creative, but unreliable. That’s what separates us from machines. We can create something new. We can think of creative ways to solve problems. Machines cannot do that, at least not yet.

On the other hand, we are incapable of repeating the same operations over and over again while producing exactly the same results. Try to do the same thing many times, and you’ll see that you will not be able. You will make mistakes. You will not be able to do exactly the same thing many times without the slightest deviation. We’re not good at that, but machines are.

Knowing that we are good at being creative and that machines are much better at performing repetitive tasks, the separation of responsibilities becomes self-evident. We should be creating new stuff or improving something. Since everything we do is expressed as code, we store everything we do in Git repositories. After that come the repetitive tasks of applying what is in Git inside our systems. All that means is that **we (humans) are defining the desired state**, while machines are making sure that the actual state becomes the same as the desired one. **Machines are converging the actual into the desired state.**

Having a single source of truth helps us separate the responsibilities between humans and machines. But that is not the only advantage. A single source of truth helps us understand the desired state of a system and find deviations between the desired and the actual state. That’s why it is of utter importance that everything, without any exception, is defined as code. The moment you SSH into a server and execute a command that changes its state, the desired state becomes unknown. From that moment on, you lose control of the actual state and whether it differs from the desired state. Your colleagues will not know what you did. Even if you are a single-person company, your memory will fade away sooner or later, and the information will be lost. From that moment on, Git stops being the source of truth.

Moving on…

If the full desired state is in Git and humans can only push changes to it, Git needs to notify machines to converge the actual into the desired state. Or, as an alternative, machines need to monitor the changes in Git. The implementations differ, and you will likely end up using both methods, depending on the tools you adopt and specific use cases. For the sake of the flow, it does not really matter whether your system gets notified of changes to the desired state through Webhooks from Git, or by pulling the information from it. What matters is that the systems find out about the changes to the desired state, and work on making the actual state the same. That’s the bare minimum.

If we want to take a step further, as we should, systems should act not only when we make changes to the desired state, but compare it continuously with the actual state. The statement should not be "I want to **deploy this release of my application**" but, instead, "I want **this release of my application to always run**, no matter what happens, forever and ever, until the end of time or, at least, until I change my mind."

By defining the desired state, we do not define what it should be right now, but what it should be from now on until our desires change. For that to work, we need to **switch from imperative to declarative formats**. Having a script that installs an application works only when that application is not already installed. The system can use it once. It cannot apply it many times. Imperative declarations prevent the system from applying only the differences between the desired and the actual state. That’s why most of us used XML in the past. That’s why we switched to JSON and why most of us tend to use YAML today. Those are all examples of declarative definitions. We stopped using shell scripts except when they are wrappers that execute tools capable of **converting declarative definitions into machine instructions**.

Still, using tools capable of converting declarative definitions into machine instructions is not enough. Or, to be more precise, it is not enough in today’s highly dynamic systems. We need more. We need to become immutable.

Being immutable means that we are not changing the state of any individual resource. Instead, we build a new VM or a container image and replace existing resources with new ones based on those images, whenever we need to make a change. For example, if we want to upgrade nodes of our Kubernetes cluster, we do NOT do that at runtime. Instead, we do rolling updates by gradually shutting down the existing nodes and creating new ones based on the image that contains the version we need. We do a similar process with our applications, except that we do not deal with VMs but containers. In either case, **everything is immutable, and everything is based on images**.

Creating new resources based on images is more reliable. We know exactly what we’ll get since the state of the resources does not change. We can create images, test them, and be assured that they will behave the same no matter whether they are running in a local environment, in staging, or in production. More importantly, in the context of GitOps, immutability simplifies the work the machines need to do to converge the actual into the desired state. There are fewer moving pieces to think about, and the system can always refer to the images whenever the actual state deviates from the desired one.

Now that we saw the outcomes of applying the GitOps principles, we might dive into a more critical question. Why do we want it in the first place?

Having a **single source of truth**, everyone in our organization can easily deduce what is running where. Given that the source of truth is stored in Git repositories, we can easily see the **history of all the past desired states and the communication** that led to those. If there is an issue, we can easily find out who did what and why. That’s what Git gives us. Every change is a commit, and every commit is stored separately. We can track changes easily. We can treat our systems as a novel and deduce the events that led us to today.

Given that Git holds the whole truth, it is the only tool that matters. We can use different IDEs, a variety of builders, various programming languages, and so on and so forth. The only tool that is the same for all and adopted by everyone is Git. That allows flexibility for each of us to use whichever tool we like for local development while having a familiar destination everyone is familiar with. If all the code is in Git, and all the essential communication (comments, PRs, etc.) is there as well, everyone in an organization can rally around it and use it. Using **Git as the hub for everything increases productivity** much more than if we spread each department’s outcomes into different locations.

Git becomes the **clear boundary between the job performed by humans, and the tasks executed by machines**. Everyone knows what needs to be done. We push changes to Git, machines apply those changes. We create the desired state, while machines are making sure that the actual state matches our desires.

Finally, having a **single source of truth is a prerequisite for continuous delivery**. If we define it as "every time we push a change to Git, the whole process is fully automated and ends with a deployment of a release in one of the environments", then it stands to reason that everything must be in Git. If it is not, doing continuous delivery would be close to impossible.

To summarize, if we accept that everything is defined as code, it is logical to say that **everything is stored in Git**. That’s where code resides. If everything is in Git, **it is the single source of truth**. **The source of truth is the desired state**. Our job (as humans) is to **keep the source of truth up-to-date**, while **machines are in change of converging the actual into the desired state.**

All that begs the question. Why isn’t everyone using GitOps?

## Why Isn’t Everyone Using GitOps?

The summary of what I was saying can be: "GitOps is great, everyone should adopt it fully." Yet, not everyone is using GitOps. As a matter of fact, the vast majority of the teams are not there yet. Why is that?

The major stumbling block is almost always the difficulty in unlearning what we perceive as best practices. Our industry is moving very fast. We might even say that it moves too quickly for the majority to follow. As a result, many are stuck in the state of being "experts" in an area of software development that is obsolete. But, since such "expertize" gives them higher positions in their companies, they have a **strong interest in maintaining the status quo**. Some groups are even actively rejecting change, especially when that change would allow a broader team to learn how to operate systems.

More importantly, GitOps is **shifting tasks towards left**, and that puts specific profiles in a perceived awkward position. It is perceived since, more often than not, the shift to the left often results in a change in how everyone works, rather than empowerment of some teams at the expense of the others.

On top of all those, there is often the issue of dealing with **legacy processes, architecture, and tooling**. Many made their careers based on the ability to "click buttons". Adopting GitOps means that UIs are reserved for viewing the systems’ state, while all the actions are defined as code. The number of buttons one can click is being reduced.

Finally, there is always the argument "GitOps is not mature; therefore, we shouldn’t adopt it." Is that truly the case, or is it a defense mechanism?

## Is GitOps Mature Enough For Everyone To Adopt It?

"GitOps is not mature!" I hear that often and all I can say is that such statements are silly. GitOps is not new. The only thing new about it is the name. The basic principles around GitOps exist from the dawn of time and were used by experienced engineers in mature companies for many years now. **The tools we need to apply GitOps principles exist for at least twenty years.** Those tools are version control systems, CI/CD, configuration management, and others. However, what is happening is that GitOps, as a new term gave rise to many new tools. Those might not be mature. Everything needs time to mature. But that does not mean that we need to adopt those that were conceived yesterday. You can do perfectly well GitOps based on tools that exist for five years or more.

All that does not mean that I am suggesting to use the "old tools". I am not. There is great value in adopting Kubernetes, ArgoCD, CodeFresh.io, Lighthouse, and many other tools. They might not all be marketed as GitOps tools, but they contribute significantly towards making it happen. They are not a requirement. We could travel ten or more years back in time and still have a very successful implementation of the GitOps principles.

There is no excuse to ignore GitOps. No one can say that it is not mature, since it exists for a long time. Only the name is new.