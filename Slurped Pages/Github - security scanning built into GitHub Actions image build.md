---
link: https://fabianlee.org/2024/09/23/github-security-scanning-built-into-github-actions-image-build/
site: "Fabian Lee : Software Engineer"
excerpt: "Github Actions provide the ability to define a build workflow, and for projects that are building an OCI (Docker) image, there are custom actions available for running the Trivy container security scanner. In this article, I will show you how to modify your GitHub Action to run the Trivy security scanner against your image, and ... Github: security scanning built into GitHub Actions image build"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - github-actions
  - security
  - scan
  - vulnerability
  - docker
  - image
slurped: 2025-03-08T07:59
title: "Github: security scanning built into GitHub Actions image build"
---
Github Actions](https://docs.github.com/en/actions) provide the ability to define a build workflow, and for projects that are building an OCI (Docker) image, there are custom actions available for running the Trivy container security scanner.

In this article, I will show you how to modify your GitHub Action to run the Trivy security scanner against your image, and then add that vulnerability report as an artifact that can assist in remediation.

## Custom action for Trivy image scanner

Trivy, the open-source scanner, has a [custom GitHub Action](https://github.com/aquasecurity/trivy-action) that can be called from within the Action definition.  This is a snippet from my example ‘[.github/workflows/github-actions-buildOCI.yml](https://github.com/fabianlee/google-hello-app-logging-multiarch/blob/main/.github/workflows/github-actions-buildOCI.yml)‘.

# define registry and image name
env:
  GH_REGISTRY: ghcr.io # Github Container Registry
  FULL_IMAGE_NAME: ${{ github.repository }} # full image name: owner/image

...
      # actions building latest image
...

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@0.24.0
        with:
          image-ref: ${{ env.GH_REGISTRY }}/${{ env.FULL_IMAGE_NAME }}:latest
          format: table
          output: trivy-scan-results.txt
          exit-code: '0' # '1' would cause pipeline to fail if vulnerabilities found
          ignore-unfixed: true
          scanners: vuln
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH,MEDIUM,LOW'

This will scan at the application library and OS level for any vulnerability from LOW to CRITICAL level.  It will not fail if it finds a vulnerability at these levels, it will just create a report named “trivy-scan-results.txt”.

## Custom action to upload report as artifact

So these vulnerabilities can be addressed and remediated, we are going upload the report created in the last step as an artifact to the Action.  This will create a persistent (90 day retention), record of the risk profile of this published image.

      - name: Upload trivy report as a Github Action artifact
        uses: actions/upload-artifact@v4
        with:
          name: trivy-report
          path: '${{ github.workspace }}/trivy-scan-results.txt'

This artifact can be seen on the details page of each build as shown below.

![](https://fabianlee.org/wp-content/uploads/2024/09/github-action-artifact.png)

## Example Reports

As an example of the vulnerability reports that are generated, here is an example when my [Dockerfile](https://github.com/fabianlee/google-hello-app-logging-multiarch/blob/main/Dockerfile) intentionally uses an older Go version in the multi-stage build and older Debian 11 (bullseye) build as a final runtime.

# Go 1.23 is latest at time of this writing, use older version
FROM golang:1.21.13-alpine3.19 AS builder
...
# bookworm(12) is latest as of this writing, use older version
FROM debian:bullseye-20240812-slim

![](https://fabianlee.org/wp-content/uploads/2024/09/trivy-github-scan-results-has-vuln.png)

Based on these results, I can remediate the vulnerabilities by using the latest Go 1.23 and a [distroless Debian 12](https://github.com/GoogleContainerTools/distroless) (bookworm) to minimize the attack surface.

# latest version of Go
FROM golang:1.23.1-alpine3.20 AS builder
...
# distroless version of latest Debian
FROM gcr.io/distroless/base-debian12

And now the vulnerability results are clean.

![](https://fabianlee.org/wp-content/uploads/2024/09/trivy-github-scan-results-clean.png)

REFERENCES

[Trivy GitHub action](https://github.com/aquasecurity/trivy-action)

[mafiaguy, how to embed Trivy into GitHub Actions](https://mafiaguy.medium.com/how-to-embed-trivy-into-github-actions-d702162297f5)

[distroless Debian 12](https://github.com/GoogleContainerTools/distroless)