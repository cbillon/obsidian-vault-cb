---
link: https://thenewstack.io/achieve-gitops-on-day-one-with-iac-automation/
byline: Rak Siva
site: The New Stack
date: 2023-12-22T17:00
excerpt: GitOps helps redefine how we manage infrastructure and application
  deployments in environments where precision, automation and transparency are
  vital.
twitter: https://twitter.com/@thenewstack
slurped: 2025-03-16T07:36
title: Achieve GitOps on Day One with IaC Automation
---

Throughout my tenure in the dynamic world of Fintech and digital onboarding, I’ve encountered three formidable challenges that consistently test the resilience of teams:

- **Misconfiguration and inconsistencies:** The consequences of configuration drift or inconsistencies across environments can be catastrophic. The need to confidently roll back to a proven state becomes a pressing concern in the face of regulatory scrutiny and operational risks.
- **Manual and error-prone processes:** Errors stemming from human oversight — misconfigurations, security vulnerabilities and the specter of costly downtime — loom large, casting a shadow over even the most well-planned projects.
- **Lack of visibility and auditability:** Questions like “Who made that change? When? Why?” can turn into inescapable puzzles, hampering our ability to meet compliance requirements and address security concerns.

Industry leaders like Spotify have harnessed the power of GitOps to confront these industry-specific challenges head-on, as Tim Hansen described in “[Everything is Code – Embracing GitOps at Spotify](https://www.youtube.com/watch?v=wdUiSbYsDiU)” at Kubecon North America last month. GitOps not only provides solutions but also empowers us to redefine how we manage infrastructure and application deployments in an environment where precision, automation and transparency are not just ideals but the cornerstones of success.

## What Is GitOps?

GitOps is a methodology that combines the power of git version control with Infrastructure as Code (IaC) and continuous delivery (CD) practices. At its core, GitOps treats your infrastructure and application configurations as code and stores them in git repositories. This single source of truth becomes the foundation for automated deployment and synchronization of your environments.

## GitOps Is Built on Several Core Principles

There are numerous articles that go in-depth into GitOps, I’ve summarized the key principles below. You can learn more about GitOps and these principles [here](https://thenewstack.io/4-core-principles-of-gitops/).

- **Declarative configuration**: GitOps relies on declarative configurations, where you define the desired state of your infrastructure and applications. This means you describe what you want, not how to get there, which promotes consistency and reduces configuration drift.
- **Version control**: Git is the backbone of GitOps. It offers version control, collaboration and auditing capabilities. Every change to your infrastructure or application is tracked, making it easy to roll back to previous states if needed.
- **Continuous delivery**: GitOps embraces the principles of continuous delivery, automating the deployment process. Whenever you make a change to your git repository, automated pipelines kick in to apply those changes to your environment. This reduces the risk of human error and ensures speedy, reliable deployments.
- **Automation**: Tools and processes are put in place to automatically synchronize the actual state of your infrastructure with the desired state in your git repository. No manual interventions are required for day-to-day operations.

## Why GitOps

Implementing a GitOps flow simplifies the responsibilities of developers and operations teams. Developers primarily focus on writing and committing code, while the operations team takes on the crucial role of maintaining and ensuring the provisioning and deployment of the solution is consistently safe and reliable for both developers and operations.

Developers benefit from this approach as they can concentrate on their core tasks of writing code and making improvements to the application. They don’t need to concern themselves with the intricacies of infrastructure provisioning or deployment processes. Their code changes, configurations and application updates are version-controlled in a git repository, making collaboration and tracking changes straightforward.

The Operations team plays a critical role in GitOps, ensuring that the infrastructure and deployment configurations specified in the git repository are consistent and align with best practices. They manage the automation and orchestration of these configurations, ensuring that updates are applied securely and reliably across various environments, such as development, staging and production. This approach enhances security, reduces human errors and provides a clear audit trail for compliance and troubleshooting purposes.

## Challenges to Getting Started with GitOps

Embracing GitOps often requires a shift in mindset from traditional infrastructure management. Teams need to adopt a declarative, version-controlled approach to define and manage their infrastructure, which may be a departure from their existing practices.

As per the diagram above, virtually everything in a GitOps workflow related to the software development and deployment process is stored in your repository:

- **Infrastructure as Code (IaC):** This involves managing and provisioning infrastructure through code rather than through manual processes. It enables teams to apply version control, collaboration, compliance and CI/CD practices to infrastructure management.
- **Configurations**: These are the settings and parameters that control the behavior of applications and infrastructure. In a GitOps workflow, configurations are version-controlled and managed alongside the code, ensuring consistency and traceability.
- **Application code:** This is the actual code of the applications being developed. In GitOps, the application code is maintained in a version control system, often alongside the IaC and configurations, to ensure a unified approach to versioning and changes.
- **CI/CD pipeline:** CI/CD pipelines define the automated steps that code changes go through from development to production. In GitOps, these pipelines are also treated as code and versioned, allowing for automated, repeatable and reliable processes.

At a glance, this seems simple enough, however, within these assets lie the true challenges of establishing a successful GitOps workflow. For example, while declaring Infrastructure as Code offers numerous benefits including version control and repeatability. However, it also comes with its own set of challenges.

- **Complexity:** Infrastructure can be complex, involving various resources like virtual machines, networks, storage and security policies. Writing code to define and manage this complexity can be challenging, especially for larger and more intricate environments.
- **Learning curve:** Adopting IaC often requires team members to learn new languages (Terraform, AWS CloudFormation, Ansible), tools and best practices. This learning curve can slow down initial implementation and lead to mistakes.
- **Testing and validation:** Ensuring that IaC templates work as expected can be difficult. Comprehensive testing and validation processes are needed to catch potential issues before deployment. This includes unit testing, integration testing and validation against target environments.
- **Infrastructure drift:** When bringing IaC to a CI/CD pipeline, teams will also have to deal with drift. Drift occurs when the requirements of your application begin to diverge from the state defined in your IaC code.

For these reasons, GitOps is particularly challenging for teams who struggle with or haven’t yet fully embraced IaC. This can be an overwhelming transition, but luckily there are approaches emerging that help make GitOps achievable without embracing IaC.

## Getting started with GitOps from Day One

To embark on your GitOps journey with confidence, it’s essential to streamline as much of the process as possible. This can be accomplished using automation and abstraction frameworks that can address the concerns associated with IaC.

Compare the following diagram to the one shown above. This flow addresses some of the concerns raised previously by using an automation framework to eliminate some of the assets that were originally manually crafted. We’re using our [open source Nitric Framework](https://nitric.io/) in this example; other tools can be used to automate pieces of this flow as well.

![](https://cdn.thenewstack.io/media/2023/12/caed22df-image2.png)

- **Local development and version control:** Developers start by scaffolding projects and writing application code in local environments that simulate cloud infrastructure. This approach enables them to iterate and test their code offline. Changes are committed to a version control repository, with developers deciding when to deploy their code to the cloud.
- **CI/CD pipeline and Nitric Framework:** When the application is ready for a higher environment (such as staging or production), the CI/CD pipeline, integrated with the Nitric framework, infers the necessary cloud infrastructure requirements from the application code. It generates a resource specification that acts as a source of truth for the infrastructure and can be version-controlled for audit purposes.
- **Infrastructure synchronization:** The Nitric framework uses the resource specification to determine and deploy the required infrastructure components to the cloud provider. This process ensures that the infrastructure is always in sync with the application code, eliminating the possibility of configuration drift each time the CI/CD pipeline runs.
- **Role of operations teams:** Even though the need for manually crafted Infrastructure as Code is reduced, operations teams still play a crucial role. They are responsible for implementing the infrastructure providers that fulfill the requirements of the application, such as scale and availability.

This approach streamlines the product development cycle, reduces time to market and fosters a collaborative environment between developers and operations teams. It effectively balances automation with control, ensuring that infrastructure changes are both efficient and well-managed.

## Leverage IaC Automation to Achieve GitOps

The prospect of constructing and deploying application code might appear daunting. By harnessing the power of modern Infrastructure as Code automation tools and embracing the GitOps methodology, businesses can find a seamless route from development to production.

Automation frameworks like Nitric focus on [streamlining time-consuming manual steps](https://nitric.io/docs/guides/getting-started/concepts) from the workflow which can place you in a position to build and release applications using a GitOps workflow from day one.

For those interested in exploring this approach further, consider [exploring Nitric](https://nitric.io/).

 Created with Sketch.