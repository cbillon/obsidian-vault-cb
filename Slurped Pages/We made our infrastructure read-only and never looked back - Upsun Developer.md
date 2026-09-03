---
link: https://developer.upsun.com/posts/how-it-works/we-made-our-infrastructure-read-only-and-never-looked-back
site: Upsun Developer
excerpt: Learn why read-only infrastructure eliminates entire classes of
  security attacks, improves reproducibility, and simplifies compliance while
  enabling predictable deployments.
slurped: 2026-08-25T09:46
title: We made our infrastructure read-only and never looked back - Upsun Developer
---

If you’ve used Upsun, you’ve likely noticed something unusual when SSH’ing into your container: the filesystem is read-only. You deploy an application, log in to investigate, attempt to create a directory, and encounter this error:

This can be frustrating at first. What kind of platform won’t let you create folders? The answer lies in a deliberate architectural decision. Read-only infrastructure provides significant security, reliability, and compliance benefits that outweigh the operational adjustments required. Let’s examine these benefits and the trade-offs involved.

## Security: eliminating entire attack vectors

Read-only filesystems eliminate entire classes of security vulnerabilities. Common attack patterns become impossible: Consider CMS vulnerabilities where attackers upload malicious executables and compromise websites. With read-only filesystems, these attacks fail at the most fundamental level. Container escape vulnerabilities that rely on writing to host filesystems—such as [CVE-2019-5736](https://nvd.nist.gov/vuln/detail/CVE-2019-5736), which allowed attackers to overwrite the host container runtime binary—cannot succeed when the filesystem rejects all write operations. Without write access, attackers cannot:

- Upload backdoors or web shells
- Modify application code to inject malicious functionality
- Install persistence mechanisms
- Alter system binaries for privilege escalation

This security boundary exists at the kernel level. Even when an attacker successfully compromises your application, the read-only filesystem significantly constrains their capabilities and prevents lateral movement.

## The trade-off: managing multiple filesystem layers

This architecture does introduce complexity. We maintain separate filesystems for `/app` (your application code) and `/` (the base operating system). We don’t rebuild your application every time we patch a Debian package on the root filesystem. This requires managing different SquashFS images for each layer. This approach demands strict ABI (Application Binary Interface) stability. Breaking the ABI would cause applications to fail. That’s why specific versions of container images remain on the same Debian version permanently. Once you’re running `type: python:3.11` with Debian 11, that combination stays locked to Debian 11 throughout its lifecycle. This constraint ensures compatibility but reduces flexibility in base system updates.

## Reproducibility: what you deploy is what you get

Upsun environments map directly to git branches. What you deploy is precisely what runs on the server. Files cannot be modified because they’re read-only. This guarantees reproducibility and integrity. When you merge code from staging to production, you’re deploying the identical filesystem image. There’s no configuration drift, no manual modifications, no “works on my machine” discrepancies. The code that passed testing is byte-for-byte identical to what runs in production.

## Explicit state management: declaring what changes

Read-only infrastructure requires explicit declaration of writable directories. Rather than allowing arbitrary filesystem writes, you specify which directories need write access for uploads, cache, or temporary data. This intentionality makes application behavior predictable and state management explicit. Configure writable mounts in your `.upsun/config.yaml`:

This configuration makes state location transparent. When you clone data from production to staging, mount contents transfer with it, enabling reliable reproduction of the complete environment—both code and state.

## Infrastructure benefits: platform-wide read-only design

The entire Upsun infrastructure operates on read-only filesystems. This eliminates an entire category of operational problems: in-place system upgrades. Distribution upgrades and live system patching are notoriously difficult. Package conflicts, dependency resolution failures, configuration file drift, and partial upgrade states create unpredictable system behavior. When managing infrastructure at scale, these issues multiply—each host becomes a unique snowflake with its own upgrade history and potential failure modes. Read-only infrastructure sidesteps this complexity entirely. We don’t run `apt-get upgrade` on live systems. We build new VM images with updated packages, test them thoroughly, and replace running VMs wholesale. This approach requires solving different technical challenges (particularly safe VM replacement under load), but once implemented, it eliminates the fragility and unpredictability inherent in in-place upgrades. The operational benefits scale significantly. When managing thousands of VMs across multiple cloud providers, knowing that every VM is identical eliminates entire classes of debugging sessions. Configuration drift between hosts becomes impossible. When functionality is verified on one VM, it functions identically on all VMs running that image.

## Compliance: simplified security scanning

Read-only infrastructure reduces compliance complexity significantly. Runtime antivirus scans on system directories become unnecessary—immutable filesystems cannot be infected with malware during operation. Security scanning shifts to build time. You catalog deployed packages and their versions, scan the filesystem image once during creation, and subsequently monitor for newly disclosed CVEs affecting your deployed versions. This eliminates continuous filesystem monitoring and runtime scanning overhead while maintaining security posture. Audit trails become more straightforward. Application code changes are tracked through git history. Infrastructure changes are documented as image replacements in your deployment pipeline. The immutability eliminates ambiguity about what changed and when, simplifying compliance reporting and incident investigation.

## Getting started with read-only infrastructure

Ready to implement these security and operational benefits? [Create a free Upsun account](https://auth.upsun.com/register) and deploy your first application with read-only infrastructure built in. For guidance on configuring writable directories or adapting applications to immutable infrastructure, consult our [documentation on writable mounts](https://docs.upsun.com/create-apps/app-reference/single-runtime-image.html#mounts) to learn how to explicitly declare directories requiring write access.