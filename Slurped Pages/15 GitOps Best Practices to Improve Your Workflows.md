---
link: https://spacelift.io/blog/gitops-best-practices
byline: Written by
site: Spacelift
excerpt: Discover the role of GitOps and learn actionable best practices that will help improve your GitOps workflows.
twitter: https://twitter.com/@handle
slurped: 2025-04-03T06:20
title: 15 GitOps Best Practices to Improve Your Workflows
tags:
  - git
  - gitops
---

GitOps is the practice of using versioned code files in Git repositories to manage the software development process. It makes Git the source of truth for your workflows, unlocking efficiency and reliability improvements.

A robust GitOps strategy is crucial to the successful adoption of other DevOps technologies, including IaC, CI/CD, and configuration management. Setting up a basic GitOps implementation can be relatively straightforward, but scaling up to larger environments is often daunting. You need to keep your repositories maintainable and prevent live deployments from drifting away from their correct states.

In this article, we’ll discuss 15 best practices for advanced GitOps implementations. We’ll cover the top techniques for enhancing GitOps workflows at scale, enabling you to build a more effective software delivery process. Let’s begin with a quick recap of GitOps.

## Advanced GitOps best practices[](https://spacelift.io/blog/gitops-best-practices#advanced-gitops-best-practices)

### 1. Fully automate all processes (apps and infrastructure)[](https://spacelift.io/blog/gitops-best-practices#1-fully-automate-all-processes-apps-and-infrastructure)

The basic definition of GitOps is simple: **The state of your resources is defined by code and config files stored in a Git repository**. Tools then process those files to populate your live environments. 

This isn’t an inherently automated workflow, though: In small teams, DevOps team members may need to periodically pull repositories and run `[terraform apply](https://spacelift.io/blog/terraform-apply)` or `kubectl apply` against them.

To make GitOps work at scale, you need to **fully automate the process**. Use CI/CD pipelines to roll out changes automatically when new commits are merged. This ensures your apps and infrastructure always match the state of your repositories. 

It also prevents the errors and conflicts that can occur when two developers manually apply changes at the same time. Moreover, you won’t need to grant developers direct access to cloud credentials.

### 2. Split projects and pipelines into composable modules[](https://spacelift.io/blog/gitops-best-practices#2-split-projects-and-pipelines-into-composable-modules)

GitOps workflows can be challenging to maintain at scale. New components added to your stack cause repositories to grow and become disorganized, while CI/CD configurations end up being duplicated to handle small differences in environments.

You can mitigate these problems by **splitting your configs into smaller reusable units**. For instance, publishing your own [Terraform module registry](https://spacelift.io/blog/terraform-registry) allows you to compose your configs from existing components. This reduces maintenance overheads and allows new resources to be assembled more efficiently. 

Avoid hardcoding values in your configurations by instead reading environment variables you can customize each time a module is used.

### 3. Decouple development, delivery, and deployment tasks[](https://spacelift.io/blog/gitops-best-practices#3-decouple-development-delivery-and-deployment-tasks)

GitOps allows you to automate your entire software lifecycle, from development through to delivery and deployment. Nonetheless, you should keep these tasks decoupled so they’re easy to govern and maintain. This means using separate repositories and pipelines for each config type:

- **Development:** Source code and IaC files should be tested and validated first.
- **Delivery:** A separate pipeline prepares changes for continuous delivery, running policy-as-code checks stored in another repository.
- **Deployment:** A continuous deployment flow handles the actual rollout to production, executing commands like `terraform apply`.

Keeping each phase distinct makes it easier to track progress, enforce access requirements, and pinpoint the causes of failures.

### 4. Use declarative configuration for every resource[](https://spacelift.io/blog/gitops-best-practices#4-use-declarative-configuration-for-every-resource)

GitOps revolves around **declarative configuration**. The configuration files in your repositories should describe _what_ you want to be deployed, not _how_ to deploy it. This makes your configs easier to understand, maintain, and audit. 

Declarative configuration is also less error-prone than traditional infrastructure provisioning scripts, where developer mistakes could inadvertently reorder important operations.

The popular tools **Kubernetes** and **Terraform** are primarily declarative in nature: You specify you want a certain infrastructure component to exist, and the platform automates the process of achieving that state in your environment.

### 5. Ensure the Git repository is the single source of truth[](https://spacelift.io/blog/gitops-best-practices#5-ensure-the-git-repository-is-the-single-source-of-truth)

GitOps only succeeds when it’s the single source of truth. This means only your repositories should define the states of your environments. Conflicts, inconsistencies, and errors easily occur when other processes are used at the same time.

For example, developers may reconfigure resources manually if they feel it’s quicker or more convenient. However, unless they’ve been committed to your repository, those changes will be reverted and lost the next time the GitOps pipeline runs.

### 6. Adopt pull-based, agent-driven deployment strategies[](https://spacelift.io/blog/gitops-best-practices#6-adopt-pullbased-agentdriven-deployment-strategies)

GitOps CI/CD pipelines give you a choice of two different high-level strategies: push or pull. 

- **Push-based CI/CD** is the traditional approach. A script running on your CI/CD server runs commands like `kubectl apply` to “push” deployments to your infrastructure.
- **Pull-based deployment** is the modern alternative. Under this model, an agent running inside your target environment periodically polls your Git repositories. The agent “pulls” the changes and automatically applies necessary actions to reconcile your environment’s state.

Compared to push-based deployment, pull-driven strategies offer automation, reliability, and security benefits. In particular, you don’t need to share infrastructure access keys with your CI/CD server, so there’s less chance of live environments being breached.

### 7. Implement immutable deployments and releases[](https://spacelift.io/blog/gitops-best-practices#7-implement-immutable-deployments-and-releases)

Immutable deployments make GitOps release processes safer and more predictable. They enable you to treat new releases as fully independent snapshots that will never change once created.

The practice means that instead of continually reusing tags like `latest` and `production`, **each release should be assigned its own unique identifier**. These IDs are typically derived from the Git commit SHA that’s being deployed. Once a release has been tagged, it should be impossible to reuse that tag or change its contents.

Immutable deployments make it easier to revert problematic changes. You can simply redeploy the last good tag instead of pushing another update to the `latest`. Immutability is also a crucial compliance tool, as it helps prove that releases haven’t been tampered with post-creation.

### 8. Configure drift detection to flag unexpected infrastructure changes[](https://spacelift.io/blog/gitops-best-practices#8-configure-drift-detection-to-flag-unexpected-infrastructure-changes)

Drift is one of the biggest issues facing infrastructure teams at scale. It’s the phenomenon that occurs when [resources gradually deviate](https://spacelift.io/blog/what-is-configuration-drift) from their expected or desired state over time. In practical terms, drift means your **live infrastructure no longer matches the configuration defined in your Git repositories,** despite your successful adoption of a robust GitOps workflow. It can cause errors, misconfigurations, and compliance breaches.

Automated drift detection scans enable you to find and fix drift as it happens. They compare your infrastructure’s current state to the configuration defined in your Git repository, using IaC tool features like [`terraform plan`](https://spacelift.io/blog/terraform-plan). 

Drift usually becomes harder to solve the longer it exists, so regular scans are crucial to ensure your infrastructure runs reliably. Drift detection and resolution capabilities are typically found in IaC orchestration platforms like [Spacelift](https://spacelift.io/).

### 9. Make GitOps workflows portable across different environments and clouds[](https://spacelift.io/blog/gitops-best-practices#9-make-gitops-workflows-portable-across-different-environments-and-clouds)

GitOps can enhance operational flexibility by standardizing multicloud deployments and enabling concurrent releases to different environments. However, this depends on your GitOps workflows being designed for portability. You should **avoid using tools and processes that are tightly coupled to specific cloud providers**, as this increases the risk of vendor lock-in.

Infrastructure configurations are inherently specific to individual clouds — the [Terraform config for an AWS EC2 instance](https://spacelift.io/blog/terraform-ec2-instance) will look different from one for GCP GCE, for example — but you can take steps to improve portability. For example, it’s often possible to split common config sections into reusable modules that reduce the amount of duplication required.

Advanced tool choices such as [Crossplane](https://www.crossplane.io/) can also help abstract differences between clouds. Crossplane enables you to provision infrastructure components using Kubernetes custom resources. It means developers can create a new `ComputeInstance` or `Database` without understanding all the cloud provider interactions involved.

### 10. Nurture a GitOps culture in your organization[](https://spacelift.io/blog/gitops-best-practices#10-nurture-a-gitops-culture-in-your-organization)

GitOps success depends on your whole team adopting it with confidence and enthusiasm. You need to nurture a mindset that ensures IaC, CI/CD, and declarative configuration are everyone’s default approach to operations.

Team members need to understand how GitOps processes work, why they’re used, and how they benefit from them. For instance, you could highlight how GitOps reduces the frequency of infrastructure incidents, giving developers more time to focus on meaningful work. 

Creating a **personal connection** makes it less likely that individuals will try to bypass GitOps and continue using legacy processes.

### 11. Use GitOps for infrastructure and configuration management, not just app deployments[](https://spacelift.io/blog/gitops-best-practices#11-use-gitops-for-infrastructure-and-configuration-management-not-just-app-deployments)

Many DevOps teams have started adopting GitOps for app deployments, such as by using tools like [Argo CD to release to Kubernetes](https://spacelift.io/blog/argocd). However, this is just one key GitOps use case — the strategy applies equally well to infrastructure management using IaC tools and configuration management via solutions like Ansible.

Aligning your entire software delivery lifecycle around GitOps improves the consistency and maintainability of your projects. It empowers you to manage all aspects of your apps and infrastructure using a single high-level automated process. Not only does this simplify operations, it also helps share skills and knowledge. 

Developers who are not specially trained in infrastructure management could safely apply urgent infrastructure changes by committing IaC files via your GitOps workflow.

### 12. Prefer tools that natively integrate with each other[](https://spacelift.io/blog/gitops-best-practices#12-prefer-tools-that-natively-integrate-with-each-other)

GitOps and related topics like IaC and CI/CD are some of the most active fields in the DevOps space. New tools appear regularly, so there are plenty of options for different use cases. 

However, it’s important to **check your platforms can seamlessly integrate with each other.** This simplifies setup tasks and supports your security posture.

Combining a major version control system, CI/CD platform, cloud provider, and IaC tool shouldn’t pose a problem, but the situation could be trickier if you’re using more specialist solutions. Make sure you audit each tool to understand how it interacts with the others and any security risks it presents. 

The benefits of native integrations can be seen in Spacelift’s connectivity [with cloud providers](https://docs.spacelift.io/integrations/cloud-providers), for instance, which dynamically generates short-lived cloud credentials to use in your IaC runs.

### 13. Use trunk-based Git branching for your GitOps repositories[](https://spacelift.io/blog/gitops-best-practices#13-use-trunkbased-git-branching-for-your-gitops-repositories)

GitOps works best with trunk-based Git branching. This refers to the practice of developing all changes using short-lived feature branches that are merged into a single “trunk” branch. It **avoids the merge conflicts and overheads** that occur when developers try to maintain and synchronize multiple adjacent branches like `main`, `develop`, and `release`.

With trunk-based branching, you can simply point your CI/CD and IaC agents at your repository’s trunk. 

Developers work on feature branches and then merge them into the trunk once all necessary approvals have been made. The agents will detect the merge and automatically apply the trunk’s current state to your live environments. This eliminates branching complexity and ensures what’s deployed is always clear.

### 14. Implement progressive delivery strategies[](https://spacelift.io/blog/gitops-best-practices#14-implement-progressive-delivery-strategies)

GitOps makes it easy to implement progressive delivery strategies such as canary and blue-green deployments. These techniques improve release safety by initially limiting new deployments to a subset of users.

Tools such as [Argo Rollouts](https://spacelift.io/blog/argo-rollouts) gradually ramp up availability until everyone’s using a new release without requiring any manual intervention. If a failure occurs, it’s possible to automatically revert to the last good state from your repository. GitOps continually provides the source of truth for what the final deployment should look like.

### 15. Enforce access and compliance requirements using policy-as-code governance[](https://spacelift.io/blog/gitops-best-practices#15-enforce-access-and-compliance-requirements-using-policyascode-governance)

GitOps workflows are powerful and convenient, but this means they also pose security and compliance threats. It’s crucial to enforce granular access controls so that **only authorized developers can commit changes**. 

Moreover, you need to be able to **govern approval conditions and prevent possible misconfigurations** from being applied.

Platforms that support policy-as-code engines like [Open Policy Agent (OPA)](https://spacelift.io/blog/what-is-open-policy-agent-and-how-it-works) provide the versatility and control needed for GitOps workflows at scale. policy-as-code lets you define precise rules that your IaC, CI/CD, and app deployment configs must meet before they’re merged. 

Because the policies themselves are code, they can also be managed as part of a GitOps workflow. This enables you to test your rules automatically before they’re enabled in your projects.