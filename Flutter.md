# Continuous Integration and Deployment in Flutter

## Contents

- [Introduction](#introduction)
- [Continuous Integration](#continuous-integration)
  - [Prerequisites](#prerequisites)
  - [Getting Started](#getting-started)
  - [Linting](#linting)
  - [Flutter Packages](#flutter-packages)
  - [Flutter Web Applications](#flutter-web-applications)
  - [Flutter Android and iOS Applications](#flutter-android-and-ios-applications)
  - [Tips and Tricks](#tips-and-tricks)
  - [Examples](#examples)
- [Continuous Deployment](#continuous-deployment)

## Introduction

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

### Linting

Linting is the process of analyzing your code for potential errors, style issues, and formatting inconsistencies. Both Flutter and Dart provide CLI commands for running static code analysis, making it easy to integrate linting into your Continuous Integration (CI) workflows. Flutter extends Dart analysis with additional checks specifically for Flutter code, such as detecting incorrect widget usage and performance issues.

#### Configuring Lint Rules

You can configure linting rules in an `analysis_options.yaml` file. This file should be in the project's root directory. (Along with `pupbspec.yaml`) Within this file, all linting rules can be configured. In order to run static analysis, we recommend using Flutter's CLI command, `flutter analyze`. This can be run both locally and as a step within your CI pipeline.

Here’s an example of an `analysis_options.yaml` file:

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    - avoid_print
```

> [!NOTE]
> For more information on configuring linter rules, you can view the [`flutter_lints` documentation](https://docs.flutter.dev/release/breaking-changes/flutter-lints-package).

#### Adding Linting to CI Workflows

To include linting as part of your CI pipeline, simply add a step to your workflow to run static analysis. You can use `flutter analyze` in your CI workflow definition. For instance, we can continue to build on the example above,

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
      - name: Run Static Code Analysis
        run: flutter analyze
      - name: Run Flutter Tests
        run: flutter test
```

### Flutter Packages

When setting up CI for a Flutter package, the focus is typically on running unit and widget tests to ensure the package behaves as expected. This section outlines how to set up these tests for Flutter packages.

#### Setup for Unit and Widget Tests

Unit and widget tests ensure the correctness of your package’s logic and UI, respectively. To run these tests within GitHub Actions, you'll need to add the necessary steps in your workflow.

1. **Unit Tests:**
   Unit tests validate the functionality of specific methods or classes in isolation. In Flutter, unit tests are written using `flutter test` and typically reside in the `test/` folder.

2. **Widget Tests:**
   Widget tests ensure that the UI behaves as expected under certain conditions. These tests are also executed using `flutter test`.

#### Writing Unit and Widget Tests

For your workflow to succeed, you need to have unit and widget tests in place. Here’s an example of how to write a simple unit and widget test in Flutter:

#### Example Unit Test (`test/unit_test.dart`):

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('Simple math test', () {
    int result = 2 + 2;
    expect(result, 4);
  });
}
```

#### Example Widget Test (`test/widget_test.dart`):

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('Counter increments smoke test', (WidgetTester tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: Text('Counter App')),
          body: Center(child: Text('0', key: Key('counter'))),
          floatingActionButton: FloatingActionButton(
            onPressed: () {},
            child: Icon(Icons.add),
          ),
        ),
      ),
    );

    expect(find.text('0'), findsOneWidget);

    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    expect(find.text('1'), findsOneWidget);
  });
}
```

In order to run these tests in CI, we can use the same workflow definition from above.

### Flutter Web Applications

When setting up Continuous Integration (CI) for Flutter web applications, you typically use a combination of unit tests, widget tests, and integration tests to ensure a well-functioning and bug-free web application. Below is an explanation of how to incorporate each type of test, followed by a complete workflow example.

#### 1. **Unit and Widget Tests**

Unit and Widget tests can be setup in a very similar way to a Flutter library and so we can use the examples above.

#### 2. **Integration Tests**

Integration tests validate the app as a whole, testing its behavior in a browser to ensure that different components interact correctly. These tests simulate user actions in a real web environment.

To write integration tests for Flutter web applications, you can use the `integration_test` package to test UI and behavior across multiple screens. You can view the Flutter documentation for more information on intergration testing.

1. To add the `integration test` and `flutter_test` packages as dev dependencies you can use the following command: `flutter pub add 'dev:integration_test:{"sdk":"flutter"}'`
2. Next, create the directory `integration_test` in the root directory of the project. You will also need to create a `test_driver` directory in the root directory.
3. Within the `test_driver` directory, create a file called `integration_test.dart` and add the following dart code:

```dart
import 'package:integration_test/integration_test_driver.dart';

Future<void> main() => integrationDriver();
```

4. You can now add integration tests within the `integration_test` directory. Examples of integration tests can be found in 2023-MarineConservationApp's `integration_test` directory.

5. In order to run the integration test's locally, you can use the following command: `flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart -d chrome`

> [!NOTE]
> You will need to have chrome installed in order to run this test.

6. In order to add this to your CI pipeline, the integration tests will need to be run as headless tests. To do this, you will need to install chromedriver and ensure it is running the background. This can be done with the following workflow step

```yaml
steps:
  ...
  - name: Start Chromedriver
    run: chromedriver --port=4444 &
```

7. After that step has been added, you can now execute `flutter drive` in a headless environment by add the following step.

```yaml
steps:
  ...
  - name: Start Chromedriver
    run: chromedriver --port=4444 &
  - name: Run Flutter Integration Tests
    run: flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart -d web-server --headless
```

#### 3. **Complete Web Application Workflow**

Putting it all together, we can create something quite comprehensive, here's a slimmed version of [2023-MarineConservationApp's](https://github.com/spe-uob/2023-MarineConservationApp/blob/dev/.github/workflows/ci.yml) CI workflow.

```yaml
test-lint-web:
  name: Test and Lint Web App
  runs-on: ubuntu-latest
  defaults:
    run:
      working-directory: "mca-web"
  steps:
    - name: Checkout Repo
      uses: actions/checkout@v4
    - name: Setup and Install Flutter
      uses: subosito/flutter-action@v2
      with:
        channel: "stable"
        cache: true
    - name: Enable Web
      run: flutter config --enable-web
    - name: Install Flutter Dependencies
      run: flutter pub get
    - name: Run Flutter Linter
      run: flutter analyze
    - name: Run Flutter Unit and Widget Tests
      run: flutter test test
    - name: Start Chromedriver for Integration Tests
      run: chromedriver --port=4444 &
    - name: Run Flutter Integration Tests
      run: flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart -d web-server --headless
```

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

### Examples

If you would like to explore a complete example of a Continuous Integration workflow for a Flutter project, you can take a look at this [GitHub repository](https://github.com/spe-uob/2023-MarineConservationApp). This repository contains multiple workflow definitions for CI that can be used as a guide.

## Continuous Deployment

TODO
