---
link: https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/
byline: Geva Perry
site: Configu - Rethinking Configuration Management
date: 2024-04-14T20:36
excerpt: Environment variables in Docker Compose are dynamic values that can
  influence the behavior of applications running inside containers.
slurped: 2024-07-01T12:02
title: 4 Ways to Set Docker Compose Environment Variables - Configu
---

[Environment variables](https://configu.com/blog/environment-variables-how-to-use-them-and-4-critical-best-practices/) in Docker Compose are dynamic values that can influence the behavior of applications running inside containers. They allow for the configuration of containerized applications in different environments without altering the Dockerfile or application code. 

In Docker Compose, environment variables can be set in several ways: directly in the docker-compose.yml file, in an external .env file, or passed through the command line. These variables are used to manage secrets, database connections, and service configurations, making it easier to deploy applications across various environments with minimal changes to the deployment scripts.

- [What Are Environment Variables in Docker Compose?](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#What_Are_Environment_Variables_in_Docker_Compose "What Are Environment Variables in Docker Compose?") 
- [4 Ways to Set Environment Variables in Docker Compose](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#4_Ways_to_Set_Environment_Variables_in_Docker_Compose "4 Ways to Set Environment Variables in Docker Compose") 
    - [1. Using an External .env File](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#1_Using_an_External_env_File "1. Using an External .env File")
    - [2. Use the Environment Attribute](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#2_Use_the_Environment_Attribute "2. Use the Environment Attribute")
    - [3. Use the env_file Attribute](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#3_Use_the_env_file_Attribute "3. Use the env_file Attribute")
    - [4. Substitute from the Shell](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#4_Substitute_from_the_Shell "4. Substitute from the Shell")
- [Understanding Environment Variables Precedence in Docker Compose](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Understanding_Environment_Variables_Precedence_in_Docker_Compose "Understanding Environment Variables Precedence in Docker Compose") 
- [Best Practices for Working with Environment Variables in Docker Compose](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Best_Practices_for_Working_with_Environment_Variables_in_Docker_Compose "Best Practices for Working with Environment Variables in Docker Compose") 
    - [Handle Sensitive Information Securely](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Handle_Sensitive_Information_Securely "Handle Sensitive Information Securely")
    - [Understand How Interpolation Works within Compose Files](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Understand_How_Interpolation_Works_within_Compose_Files "Understand How Interpolation Works within Compose Files")
    - [Use Command Line Overrides](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Use_Command_Line_Overrides "Use Command Line Overrides")
- [Configu: The Right Way to Manage Environment Variables in Docker Compose](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Configu_The_Right_Way_to_Manage_Environment_Variables_in_Docker_Compose "Configu: The Right Way to Manage Environment Variables in Docker Compose")
    - [Configu Orchestrator](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Configu_Orchestrator "Configu Orchestrator")
    - [Configu Cloud](https://configu.com/blog/4-ways-to-set-docker-compose-environment-variables/#Configu_Cloud "Configu Cloud")

## 4 Ways to Set Environment Variables in Docker Compose 

Here are the primary ways to set environment variables in Docker Compose.

### 1. Using an External `.env` File

A `.env` file is a simple, key-value paired file used to store environment variables. By placing a `.env` file in the same directory as your `docker-compose.yml`, Docker Compose automatically picks up the variables defined within it. 

This method is particularly useful for setting up project-wide configurations that apply to multiple services or containers. It simplifies management by centralizing environment variables in a single file, making them easily editable and distributable across team members.

The use of an `.env` file also helps in keeping sensitive information out of version control by excluding the file from source code repositories. Developers can create an `.env.example` file with dummy values to share the required structure without compromising security.

### 2. Use the Environment Attribute

Within the `docker-compose.yml` file, the environment attribute allows you to define environment variables directly. These variables are then passed into the container at runtime. This method is straightforward and keeps configurations alongside their respective services in the compose file. It’s useful for setting service-specific variables or for overriding defaults provided in an .env file for specific containers.

However, directly including sensitive information in `docker-compose.yml` files is not recommended due to security risks associated with storing such data in version control systems. In such cases, referencing an external file or using Docker secrets for sensitive data is preferred.

Here is an example showing how to set an environment variable via the `environment` attribute. Below is a snippet from a `docker-compose.yml` file, which defines an environment variable `DEBUG` with a value of `false`, under the `environment` attribute for a service named `webapp`. The variable is then accessible to applications within the container.

```
...
services:
  webapp:
    image: example/webapp:latest
    environment:
      - DEBUG=false
```

### 3. Use the `env_file` Attribute

The `env_file` attribute in the `docker-compose.yml` file specifies the path to a file containing environment variables. This method allows for a clean separation between service configuration and environment variables, enabling the latter to be stored in a separate file. It’s suitable for specifying service-specific environment variables without cluttering the `docker-compose.yml` file.

This approach supports maintaining different environment variable files for different environments (e.g., development, testing, production), allowing for easy switching and management of environment-specific configurations.

For example, below is a snippet from a `docker-compose.yml` file with an `env_file` attribute that specifies a file named `webapp_env.list`. Docker Compose automatically loads environment variables from this file into the `webapp` service at runtime:

```
...
services:
  webapp:
    image: example/webapp:latest
    env_file: 
      - webapp_env.list
```

### 4. Substitute from the Shell

Environment variables can be passed directly from the shell when running Docker Compose commands. This method is useful for temporary overrides or for passing sensitive information that shouldn’t be stored in files. Variables set in this manner take precedence over those defined in files or the `docker-compose.yml` itself, providing a way to dynamically alter container behavior without modifying configuration files.

This approach also offers flexibility for CI/CD pipelines, where environment-specific variables can be injected at runtime without altering the application’s base configuration files.

For example, this command shows how to pass the `DEBUG` environment variable with a value of `true` from the shell when starting Docker Compose:

```
DEBUG=true docker-compose up
```

## Understanding Environment Variables Precedence in Docker Compose 

The precedence of each environment variable follows a specific order that determines which value is used when variables are defined in multiple places. The precedence is as follows (#1 has the highest precedence, meaning it overrides all others, #2 overrides #3-7, etc):

1. Environment variable set using `docker compose run -e` in the CLI
2. Environment variable substituted from the shell
3. Environment variable set using the environment attribute in the `docker-compose.yml` file
4. Environment variable set using `--env-file` argument in the CLI
5. Environment variable set using `env_file` attribute in the `docker-compose.yml` file
6. Environment variable set using an `.env` file placed at base of your project directory
7. Environment variable set in a container image using the `ENV` directive. 

This hierarchy ensures that developers have the flexibility to override default values in a controlled manner. For example, a variable set using the environment attribute the `docker-compose.yml` file can be overridden by passing an environment variable in a shell command, allowing for different configurations in development, staging, and production environments without changing the core compose file.

**_Related content: Read our guide to_** [**_docker environment variables_**](https://configu.com/blog/docker-environment-variables-arg-env-using-them-correctly/)

## Best Practices for Working with Environment Variables in Docker Compose 

### Handle Sensitive Information Securely

When dealing with sensitive information such as passwords or API keys, it’s crucial to avoid storing them directly in the `docker-compose.yml` file or in version-controlled `.env` files. Instead, use Docker secrets or external secrets management tools to securely manage and access sensitive data. This practice helps in minimizing the risk of exposing confidential information and ensures compliance with security best practices.

For non-sensitive configuration, using `.env` files or the `env_file` attribute can simplify environment management without significant security concerns. However, always exclude sensitive data from these files, opting for secure storage mechanisms instead.

### Understand How Interpolation Works within Compose Files

Docker Compose supports variable interpolation within the `docker-compose.yml` file, allowing for dynamic configurations. This feature enables the use of variables defined in .env files or passed through the environment to construct dynamic values for your service configurations. Learn more about interpolation in the [official documentation](https://docs.docker.com/compose/compose-file/12-interpolation/).

However, be cautious with interpolation to ensure that your configurations remain clear and maintainable. Avoid overly complex expressions that can make your compose files difficult to read and understand.

### Use Command Line Overrides

The Docker Compose command line interface provides options to override environment variables and configuration directly from the command line. This capability is useful for temporary adjustments or for scenarios where script automation is required. Leveraging command line overrides can enhance the flexibility of your deployment process, allowing for quick adjustments without the need for configuration file changes.

While command line overrides are powerful, they should be used judiciously to keep deployments consistent and maintainable across different environments.

## Configu: The Right Way to Manage Environment Variables in Docker Compose

Configu is a configuration management platform comprised of two main components, the stand-alone Orchestrator, which is open source, and the Cloud, which is a SaaS solution:

### Configu Orchestrator

As applications become more dynamic and distributed in microservices architectures, configurations are getting more fragmented. They are saved as raw text that is spread across multiple stores, databases, files, git repositories, and third-party tools (a typical company will have five to ten different stores).

The [Configu Orchestrator](https://github.com/configu/configu), which is open-source software, is a powerful standalone tool designed to address this challenge by providing configuration orchestration along with Configuration-as-Code (CaC) approach.

### Configu Cloud

[Configu Cloud](https://app.configu.com/) is the most innovative store purpose-built for configurations, including environment variables, secrets, and feature flags. It is built based on the Configu configuration-as-code (CaC) approach and can model configurations and wrap them with unique layers, providing collaboration capabilities, visibility into configuration workflows, and security and compliance standardization.

Unlike legacy tools, which treat configurations as unstructured data or key-value pairs, Configu is leading the way with a Configuration-as-Code approach. By modeling configurations, they are treated as first-class citizens in the developers’ code. This makes our solution more robust and reliable and also enables Configu to provide more capabilities, such as visualization, a testing framework, and security abilities.

[**Learn more about Configu**](https://configu.com/)