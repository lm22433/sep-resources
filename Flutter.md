# Continuous Integration and Deployment in Flutter

## Contents

- [Introduction](#introduction)
- [Continuous Integration](#continuous-integration)
  - [Prerequisites](#prerequisites)
  - [Getting Started](#getting-started)
  - [Flutter Packages](#flutter-packages)
  - [Flutter Web Applications](#flutter-web-applications)
  - [Flutter Android and iOS Applications](#flutter-android-and-ios-applications)
  - [Tips and Tricks](#tips-and-tricks)
- [Continuous Deployment](#continuous-deployment)

### Introduction

Continuous Integration and Continuous Deployment (CI/CD) automate the processes of testing, building, and deploying applications. GitHub Actions allows developers to set up CI/CD pipelines directly in GitHub repositories, making it an efficient and accessible development tool. In this guide, we will set up a CI/CD pipeline for various different types of Flutter projects.

## Continuous Integration

### Prerequisites

### Getting Started

1. In your project's root folder, navigate to the `.github/workflows/` directory. If it doesn't exist, create it. To do this you can use your favourite file explorer or use the `mkdir` command.
2. Inside this folder, you will want to create a new YAML file. Some common file names include: `flutter.yml`, `ci.yml`, and `continuous-integration.yml`. This file will define the GitHub workflow that will serve as your CI/CD pipeline. You can take a look at the [GitHub documentation](https://docs.github.com/en/actions/writing-workflows) on workflows for more information.

Now that we have successfully create a file in the correct location, we can begin actually defining our workflow. Here is a very basic example to get stated:

```yaml
name: Continuous Integration - Flutter

on:
  pull_request:
    branches:
      - main
      - dev

jobs:
  test:
    runs-on: ubuntu-linux
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: Setup and Install Flutter
        uses: subosito/flutter-action@v2
      - name: Install Flutter Dependencies
        run: flutter pub get
      - name: Run Flutter Tests
        run: flutter test
```

This YAML file defines a simple GitHub Actions workflow for Continuous Integration (CI) in a Flutter project. The workflow is triggered when a **pull request** is opened or updated on the `main` or `dev` branches of the repository. It defines a single job that will run on an `ubuntu-linux` GitHub runner instance. This jobs has multiple steps that will checkout the repository, install Flutter, install your project's dependecies and then run Flutter's test framework. These are done sequentially.

> [!WARNING]
> In many cases, repositories may be monolithic, meaning they contain multiple projects or services (e.g., backend, frontend, mobile apps) in different directories. If your Flutter project is not located in the root directory of the repository, you will need to specify the working directory in the relevant steps. This can be done by specifying the `working-directory` option within the job scope. For example:

```yaml
jobs:
  test:
    runs-on: ubuntu-linux
    defaults:
      run:
        working-directory: "./path-to-flutter-project"
    steps: ...
```

### Flutter Packages

### Flutter Web Applications

### Flutter Android and iOS Applications

### Tips and Tricks

As a repository within the Software Engineering Project GitHub Enterprise organization, you're allocated a specific number of GitHub runner minutes per month. To optimise usage and keep execution times fast, here are a few tips:

1. **Caching Dependencies and Files:** Use caching to speed up workflow execution by reusing dependencies and the Flutter installation itself:

```yaml
- name: Setup and Install Flutter
  uses: subosito/flutter-action@v2
  with:
    channel: "stable"
    cache: true
```

> [!NOTE]
> Caching takes up repository storage space, but it helps in reducing the workflow time significantly.

2. **Parallel Jobs:** By defining separate `jobs` under the jobs section, they can be executed in parallel, which speeds up the CI process. Here's an example of splitting build and test jobs:

3. **Efficient Resource Usage:** Set up automatic checks to ensure unnecessary jobs don't run on certain conditions. For example, skip a job if only documentation files were changed.

4. **Choosing the Right Runner (macOS, Linux, Windows):** GitHub Actions provides three types of virtual environments (or "runners") for executing workflows: macOS, Linux, and Windows. It's important to choose the right runner for your needs, as it affects performance and costs.

   - **Linux Runners (`ubuntu-latest`):** Linux is generally the fastest and most cost-effective option. It’s well-suited for most Flutter CI/CD pipelines, especially for testing and building web or Android applications.
   - **macOS Runners (`macos-latest`):** macOS runners are required for building and testing iOS applications due to Xcode dependencies. However, macOS runners are billed at a significantly higher rate compared to Linux runners. macOS runners consume 10 times the compute minutes compared to Linux runners, so they can quickly deplete your allocated minutes if not managed carefully. Use macOS runners only when absolutely necessary (e.g., for iOS builds).
   - **Windows Runners (`windows-latest`):** Windows runners are necessary if you need to test or build applications in a Windows-specific environment. In most cases for Flutter development, Windows runners are less commonly needed, but they could be used for testing Windows desktop applications. Like macOS, Windows runners consume more compute minutes than Linux but are cheaper than macOS (around 2x the minutes consumed compared to Linux).

   You can specify the type of runner changing the `runs-on` value:

   ```yaml
   jobs:
     test:
       runs-on: ubuntu-latest # Change this value
   ```

When handling sensitive information such as API keys, Firebase tokens, or other credentials, store these as GitHub Secrets. You can configure secrets in your repository by navigating to Settings > Secrets. Then, reference these secrets in the workflow file as follows:

You can also use environment variables to manage values shared across steps. Define them under env and reference them directly within steps:

```yaml
env:
  FLUTTER_CHANNEL: stable

steps:
  - name: Set up Flutter
    uses: subosito/flutter-action@v2
    with:
      channel: ${{ env.FLUTTER_CHANNEL }}
```

## Continuous Deployment

TODO
