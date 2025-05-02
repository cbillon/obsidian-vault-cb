---
link: https://joachim8675309.medium.com/devops-concepts-snowflake-vs-phoenix-845e56006ccc
byline: Joaquín Menchaca (智裕)
site: Medium
date: 2024-08-30T01:41
excerpt: "The Snowflake Server pattern and Phoenix Server pattern are important
  concepts around server fleet management. These approaches revolve around how
  to manage configuration drift: A snowflake server is…"
twitter: https://twitter.com/@Joachim8675309
slurped: 2025-03-13T09:48
title: "DevOps Concepts: Snowflake vs Phoenix - Joaquín Menchaca (智裕) - Medium"
---

## Summary of Snowflake vs Phoenix Server Patterns


The _Snowflake Server pattern_ and _Phoenix Server pattern_ are important concepts around server fleet management. These approaches revolve around how to manage [**configuration drift**](http://kief.com/configuration-drift.html):

> “_The phenomenon where servers in an infrastructure become more and more different from one another as time goes on, due to manual ad-hoc changes and updates, and general entropy_” — **Kief Morris**

## Snowflake Server

A _snowflake server_ is a server that has a unique configuration that cannot be easily replicated. Over time, the server accumulates manual configurations, tweaks, and updates that make it difficult to recreate or replace. This uniqueness leads to problems with scalability, reliability, and maintainability.

_Snowflake servers_ are fragile like a real snowflake that is delicate and unique. When the _snowflake server_ fails, it is difficult to impossible to restore it to the exact previous state without manual intervention.

> “The true fragility of snowflakes, however, comes when you need to change them. Snowflakes soon become hard to understand and modify. Upgrades of one bit software cause unpredictable knock-on effects” — Martin Fowler

## Phoenix Server

A _phoenix server_, like its metaphor, arises from its ashes, where the servers are destroyed and rebuilt from scratch using an automated process. The configuration is fully scripted and version-controlled, which facilitates the ability to recreate an identical server from the same configuration by running the appropriate scripts.

_Pheonix servers_ are resilient because they are designed to be replaced rather than repaired. When a Phoenix server encounters an issue, it is simply destroyed and recreated rather than manually fixed.

> “A server should be like a phoenix, regularly rising from the ashes.” — Martin Fowler

## Fleet Management through the Ages

The evolution of fleet management has seen a shift from managing unique _Snowflake servers_ to automating the deployment and maintenance of _Phoenix servers_ using _Infrastructure as Code_ (IaC). This progression reflects the broader transition from the **Iron Age** to the **Cloud Age**.

In the **Iron Age**, physical servers, often referred to as “iron” or “bare metal,” were the primary method for hosting and running applications.

In contrast, the **Cloud Age** introduces virtualized infrastructure, where computing resources like virtual machines and containers are easily provisioned, scaled, and managed through cloud service providers such as AWS (2006), Google Cloud (2008), and Azure (2014).

## The Iron Age

In the **Iron Age**, _snowflake servers_ often required manual setup and maintenance, making them difficult to replicate or scale. However, opportunities for automation emerged with system provisioning tools like [**Kickstart**](https://wikipedia.org/wiki/Kickstart_\(Linux\)) (1999) and [**Preseed**](https://wikipedia.org/wiki/Preseed) (2005), as well as configuration management platforms like [**CFEngine**](https://cfengine.com/) (1993), [**Puppet**](https://www.puppet.com/) (2005), and [**Chef**](https://www.chef.io/products/chef-infra) (2009).

Provisioning tools helped establish a baseline configuration, such as installing an SSH key for deployments. Configuration management platforms then ensured servers adhered to a desired state, a process known as convergence, using automation scripts like Puppet Manifests or Chef recipes.

Despite these advancements, automated configuration tools only managed parts of a system’s state, leaving gaps. As Kief Morris observes:

> _“The challenge with automated configuration tools is that they only manage a subset of a machine’s state. Writing and maintaining manifests/recipes/scripts is time consuming, so most teams tend to focus their efforts on automating the most important areas of the system, leaving fairly large gaps.”_

Over time, these untracked states could drift, leading to reliability issues and security vulnerabilities. For instance, a malicious SSH binary with a backdoor could be installed, or a vulnerable SSH configuration could be added without detection.

In many environments, automation scripts reflected only what had been spot-tested or encountered during outages, often building from an unknown but functional state rather than from scratch. This approach lowered reliability and security, making the _Phoenix server_ pattern, which emphasizes rebuilding servers from a known, clean state, a better alternative.

## The Cloud Age

**The Cloud Age** introduced new ways to provision and scale server fleets using the _Phoenix Server_ pattern. This approach launches systems from base images (baked with tools like [**Packer**](https://www.packer.io/)) and configures them with automation tools such as [**cloud-init**](https://cloud-init.io/) (2008). Further adjustments, or frying, are managed by change configuration platforms like [**Puppet**](https://www.puppet.com/) (2005), [**Chef**](https://www.chef.io/products/chef-infra) (2009), [**Salt Stack**](https://saltproject.io/) (2011), and [**Ansible**](https://www.ansible.com/) (2012)

Base images are built using automation tools that incorporate system provisioning techniques, ensuring consistency across deployments. Popular autoscaling solutions like AWS [**Auto Scaling Groups**](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html) ([**ASG**](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)), Google [**Managed Instance Groups**](https://cloud.google.com/compute/docs/quickstart-mig) ([**MIG**](https://cloud.google.com/compute/docs/quickstart-mig)), and Azure [**Virtual Machine Scale Sets**](https://learn.microsoft.com/azure/virtual-machine-scale-sets/overview) ([**VMSS**](https://learn.microsoft.com/azure/virtual-machine-scale-sets/overview)) efficiently manage these systems.

The key takeaway is the fully automated process from image creation to system configuration. When systems fall out of compliance, they are automatically replaced with new instances, embodying the _Phoenix Server_ pattern, where systems are rebuilt from scratch.

## Immutable Infrastructure

Immutable infrastructure is built on the _Phoenix Server_ pattern, where servers are consistently destroyed and rebuilt from scratch using pre-baked images containing most of the configuration and state. This approach ensures that any mutable state, such as dynamic database addresses, can be injected during deployment, maintaining a clean and consistent environment.

Initially, cloud autoscaling solutions like [**VMSS**](https://learn.microsoft.com/azure/virtual-machine-scale-sets/overview), [**MIG**](https://cloud.google.com/compute/docs/quickstart-mig), and [**ASG**](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html) were used to deploy immutable infrastructure. These technologies manage fleets by leveraging images with embedded configurations, minimizing the need for in-place modifications and reducing configuration drift.

## Rise of the Containerization

The rise of containerization further solidified the concept of immutable infrastructure, making it ubiquitous. Containers encapsulate configuration within the images, simplifying deployment to running containers. This evolution led to advanced orchestration platforms like [**Kubernetes**](https://kubernetes.io/) (2014), [**Swarm**](https://docs.docker.com/engine/swarm/) (2014), and [**Nomad**](https://www.nomadproject.io/) (2015), which extended the capabilities of immutable infrastructure.

These platforms notably support stateful applications — a feature not straightforward in cloud autoscaling solutions like [**ASG**](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html), [**MIG**](https://cloud.google.com/compute/docs/quickstart-mig), or [**VMSS**](https://learn.microsoft.com/azure/virtual-machine-scale-sets/overview) — making them essential for managing distributed databases and other stateful services.

Automation tools like [**Packer**](https://www.packer.io/) (2013) are crucial in this ecosystem, enabling the creation of images used both by cloud autoscaling solutions and container orchestration platforms, ensuring consistent and reliable deployments across diverse environments.

## Conclusion

Whether you’re using bare metal servers or cloud infrastructure, adopting the _Phoenix server_ pattern — where systems are regularly destroyed and rebuilt from scratch — offers the highest reliability and lowest maintenance costs for both immutable and mutable infrastructures. While today’s automation tools make this approach cost-effective, success hinges on assembling a team with the necessary expertise in design patterns and relevant technologies.

Implementing these advanced infrastructure patterns is not just about having the right tools; it requires a team proficient in designing, deploying, and maintaining these systems. The effectiveness of the _Phoenix server pattern_ and other modern practices relies on a deep understanding of infrastructure design patterns, continuous integration and delivery pipelines, and security best practices.

## Infrastructure as Code

Modern automation practices leverage _Infrastructure as Code_ (IaC), with scripts stored in version control systems to ensure changes can be tested, scanned for security vulnerabilities, and peer-reviewed. In this context, distinct _continuous integration_ and _continuous delivery_ (CI/CD) pipelines for both the infrastructure and application layers are crucial. These pipelines should also adhere to design principles like separation of concerns and single responsibility.

## Service Discovery and Service Mesh

_Service discovery_ is essential in dynamic infrastructures, enabling services to locate each other and automatically recover through failover when necessary. Tools like [**Consul**](https://www.consul.io/), [**Eureka**](https://github.com/Netflix/eureka), and Kubernetes’ built-in service discovery facilitate this.

For secure and advanced networking, _service meshes_ like [**Linkerd**](https://linkerd.io/) and [**Istio**](https://istio.io/) manage secure service-to-service communication for microservices and other distributed clusters. These technologies support traffic splitting, which is vital for blue-green and canary deployments, allowing new application versions to run alongside existing ones to minimize downtime and reduce risk. Chaos Engineering tools like [**Chaos Monkey**](https://netflix.github.io/chaosmonkey/) further test the resilience of this infrastructure by intentionally causing failures, ensuring _Phoenix servers_ can handle real-world disruptions effectively.

## Pets vs Cattle

A related concept I’ve previously discussed is the [**Pets vs. Cattle**](https://joachim8675309.medium.com/devops-concepts-pets-vs-cattle-2380b5aab313) analogy, which compares disposable infrastructure to _cattle_ and irreplaceable infrastructure to _pets_. This closely aligns with the _Phoenix server_ pattern, where servers are treated as replaceable units, regardless of whether they operate within mutable or immutable infrastructure.

However, this analogy diverges when considering _Snowflake servers_. While _Snowflake servers_ are not inherently disposable, they can still benefit from automation that realigns their configuration to a desired state and repairs them as needed, thereby bringing some of the benefits of fleet management to traditionally unique servers.

## Final Thoughts

As infrastructure continues to evolve, the principles of automation, immutability, and resilience become increasingly central to maintaining reliable, scalable, and cost-effective systems. Whether managing a cloud-based fleet or traditional bare metal servers, understanding and applying these patterns is key to navigating the complex landscape of modern infrastructure.

## Previous Article

A _snowflake server_ is built by configuring running systems, a process called _frying_, while _phoenix servers_ are built by either configure the systems from scratch, still _frying_, or embedding the configuration in an image, a process called _baking_.

## References

- [**Phoenix Server**](https://martinfowler.com/bliki/PhoenixServer.html) by Martin Fowler on July 10, 2012
- [**Snowflake Server**](https://martinfowler.com/bliki/SnowflakeServer.html) by Martin Fowler on July 10, 2012
- [**Immutable Server**](https://martinfowler.com/bliki/ImmutableServer.html) by Kief Morris on June 13, 2013
- [**Configuration Drift**](http://kief.com/configuration-drift.html) on December 6, 2011
- [**Moving to the Phoenix Server Pattern — Part 1: Introduction**](https://www.thoughtworks.com/insights/blog/moving-to-phoenix-server-pattern-introduction) by Ama Asare on July 15, 2015
- [**Moving to the Phoenix Server Pattern — Part 2: Things to Consider**](https://www.thoughtworks.com/insights/blog/moving-to-phoenix-server-pattern-things-to-consider) by Pablo Porto on September 15, 2015
- [**Moving to the Phoenix Server Pattern Part 3: Evolving a Recovery Strategy**](https://www.thoughtworks.com/insights/blog/moving-using-phoenix-pattern-part-3-evolving-recovery-strategy) by Laura Ionescu on February 26, 2016
- Kubernetes for Serverless Applications by Russ McKendrick
- [**Configuration Drift: Phoenix Server vs Snowflake Server Comic**](https://www.digitalocean.com/community/tutorials/configuration-drift-phoenix-server-vs-snowflake-server-comic) by Digital Ocean on March 26, 2018
- [**Server Fleet Management at Scale: AWS Implementation Guide**](https://s3.amazonaws.com/solutions-reference/server-fleet-management-at-scale/latest/server-fleet-management-at-scale.pdf) by Rob Barnes in June 2018