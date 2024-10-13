# Continuous Integration and Deployment in Flutter

## Contents

- [Introduction](#introduction)
- [Continuous Integration](#continuous-integration)
  - [Getting Started](#getting-started)
  - [Linting](#linting)
  - [Flutter Packages](#flutter-packages)
  - [Flutter Web Applications](#flutter-web-applications)
  - [Flutter Android and iOS Applications](#flutter-android-and-ios-applications)
  - [Tips and Tricks](#tips-and-tricks)
  - [Examples](#examples)

## Introduction

Continuous Integration and Continuous Deployment (CI/CD) automate the processes of testing, building, and deploying applications. GitHub Actions allows developers to set up CI/CD pipelines directly in GitHub repositories, making it an efficient and accessible development tool. In this guide, we will set up a CI/CD pipeline for various Flutter projects.

## Continuous Integration

### Getting Started

1. In your project's root folder, navigate to the `.github/workflows/` directory. If it doesn't exist, create it. To do this you can use your favourite file explorer or use the `mkdir` command.

2. Inside this folder, you will want to create a new YAML file. Some common file names include: `flutter.yml`, `ci.yml`, and `continuous-integration.yml`. This file will define the GitHub workflow serving as your CI/CD pipeline. You can look at the [GitHub documentation](https://docs.github.com/en/actions/writing-workflows) on workflows for more information.

3. Now that we have successfully created a file in the correct location, we can begin defining our workflow. Here is a very basic example to get stated:

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

This YAML file defines a simple GitHub Actions workflow for Continuous Integration (CI) in a Flutter project. The workflow is triggered when a **pull request** is opened or updated on the `main` or `dev` branches of the repository. It defines a single job that will run on an `ubuntu-linux` GitHub runner instance. This job has multiple steps that will checkout the repository, install Flutter, install your project's dependencies and then run Flutter's test framework. These are done sequentially.

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

You can configure linting rules in an `analysis_options.yaml` file. This file should be in the project's root directory. (Along with `pubspec.yaml`) Within this file, all linting rules can be configured. To run static analysis, we recommend using Flutter's CLI command, `flutter analyze`. This can be run both locally and as a step within your CI pipeline.

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

To run these tests in CI, we can use the same workflow definition from above.

### Flutter Web Applications

When setting up Continuous Integration (CI) for Flutter web applications, you typically use a combination of unit tests, widget tests, and integration tests to ensure a well-functioning and bug-free web application. Below is an explanation of how to incorporate each type of test, followed by a complete workflow example.

#### 1. **Unit and Widget Tests**

Unit and Widget tests can be set up in a very similar way to a Flutter library so we can use the examples above.

#### 2. **Integration Tests**

Integration tests validate the app as a whole, testing its behaviour in a browser to ensure that different components interact correctly. These tests simulate user actions in a real web environment.

To write integration tests for Flutter web applications, you can use the `integration_test` package to test UI and behaviour across multiple screens. You can view the Flutter documentation for more information on integration testing.

1. To add the `integration test` and `flutter_test` packages as dev dependencies you can use the following command: `flutter pub add 'dev:integration_test:{"sdk":"flutter"}'`
2. Next, create the directory `integration_test` in the root directory of the project. You will also need to create a `test_driver` directory in the root directory.
3. Within the `test_driver` directory, create a file called `integration_test.dart` and add the following dart code:

```dart
import 'package:integration_test/integration_test_driver.dart';

Future<void> main() => integrationDriver();
```

4. You can now add integration tests within the `integration_test` directory. Examples of integration tests can be found in 2023-MarineConservationApp's `integration_test` directory.

5. To run the integration tests locally, you can use the following command: `flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart -d chrome`

> [!NOTE]
> You will need to have Chrome installed to run this test.

6. To add this to your CI pipeline, the integration tests will need to be run as headless tests. To do this, you will need to install Chromedriver and ensure it is running in the background. The `ubuntu-latest` GitHub Runner has Chromedriver preinstalled, therefore it's as simple as starting Chromedriver in the background:

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

We have already explored how to write unit and integration tests in the previous sections. Setting up unit tests in GitHub Actions can be done similarly to how they have been done in previous sections, as they don’t rely on platform-specific configurations. However, when it comes to running integration tests, they pose a bit more of a challenge.

Integration tests typically need to run on actual devices or emulators. This means that you need to set up emulators or simulators, handle device boot-up, and ensure that tests run smoothly across both Android and iOS platforms. This is where configuration becomes a bit more complex, especially in a CI/CD environment like GitHub Actions.

To handle these challenges, we need to configure Android emulators and iOS simulators in the CI pipeline, making sure the required dependencies are installed and the emulators are properly booted before executing the integration tests.

#### iOS

Running iOS integration tests in GitHub Actions is more complex compared to Android due to platform restrictions. Since iOS apps need to be run and tested on a macOS environment, GitHub Actions uses macOS runners to facilitate this.

**Key steps:**

1. Use macOS runner: To run iOS tests, the job must be executed on macos-latest.

2. Boot iOS Simulator: The iOS simulator is booted using Xcode’s xcrun command, which controls and manages available simulators.

3. Run integration tests: The integration tests are run using the flutter drive command, targeting the iOS simulator.

> [!CAUTION]
> MacOS GitHub runners are billed at 10x the rate of an Ubunutu GitHub runner. Please be careful when setting up CI for your iOS apps.

**Example iOS Workflow for Github Actions**

```yaml
ios-test:
  runs-on: macos-latest
  steps:
    - name: Checkout Repository
      uses: actions/checkout@v4
    - name: Setup and Install Flutter
      uses: subosito/flutter-action@v2
      with:
        channel: "stable"
        cache: true
    - name: Enable iOS
      run: flutter config --enable-ios
    - name: Install Flutter Dependencies
      run: flutter pub get
    - name: Run Flutter Static Code Analysis
      run: flutter analyze
    - name: Run Flutter Unit and Widget Tests
      run: flutter test test
    - name: Run Flutter Integration Tests
      run: |
 xcrun simctl boot "iPhone 16"
 flutter test integration_test -d "iPhone 16"
    - name: Build Flutter App
      run: flutter build ios --no-codesign
```

#### Android

Running Android integration tests in GitHub Actions is somewhat easier than iOS, as Android development tools are better supported in CI environments. You will need to install the Android SDK, set up an Android emulator, and ensure that the tests are run using Flutter's test framework.

**Key Steps:**

3. Install Flutter and dependencies: Set up Flutter and install the necessary dependencies using flutter pub get.

4. Launch Android Emulator: Use the reactivecircus/android-emulator-runner action to start and boot an Android emulator.

5. Run integration tests: Once the emulator is booted, run the integration tests using the flutter test command.

**Example Android Workflow for GitHub Actions:**

```yaml
android-test:
  runs-on: ubuntu-latest
  steps:
    - name: Checkout Repository
      uses: actions/checkout@v4
    - name: Setup and Install Flutter
      uses: subosito/flutter-action@v2
      with:
        channel: "stable"
        cache: true
    - name: Enable Android
      run: flutter config --enable-android
    - name: Install Flutter Dependencies
      run: flutter pub get
    - name: Run Flutter Static Code Analysis
      run: flutter analyze
    - name: Run Flutter Unit and Widget Tests
      run: flutter test test
    - name: Run Flutter Integration Tests
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: 34
        arch: x86_64
        profile: pixel_6_pro
        script: flutter test integration_test
    - name: Build Flutter App
      run: flutter build apk --debug
```

### Tips and Tricks

**Optimising Workflows**

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
   - **macOS Runners (`macos-latest`):** macOS runners are required for building and testing iOS applications due to Xcode dependencies. However, macOS runners are billed at a significantly higher rate compared to Linux runners. macOS runners consume 10 times the compute minutes compared to Linux runners, so they can quickly deplete your allocated minutes if not managed carefully. Use macOS runners only when necessary (e.g., for iOS builds).
   - **Windows Runners (`windows-latest`):** Windows runners are necessary if you need to test or build applications in a Windows-specific environment. In most cases for Flutter development, Windows runners are less commonly needed, but they could be used for testing Windows desktop applications. Like macOS, Windows runners consume more compute minutes than Linux but are cheaper than macOS (around 2x the minutes consumed compared to Linux).

You can specify the type of runner changing the `runs-on` value:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest # Change this value
```

**Secrets and Environment Variables**

When handling sensitive information such as API keys, Firebase tokens, or other credentials, store these as GitHub Secrets. You can configure secrets in your repository by navigating to Settings > Secrets. Then, reference these secrets in the workflow file as follows:

You can also use environment variables to manage values shared across steps. Define them under env and reference them directly within the steps:

```yaml
env:
  FLUTTER_CHANNEL: stable

steps:
  - name: Set up Flutter
    uses: subosito/flutter-action@v2
    with:
      channel: ${{ env.FLUTTER_CHANNEL }}
```

**Matrix Jobs**

Matrix jobs allow you to test your application across different versions, environments, or configurations in parallel. This is especially useful when you want to verify compatibility with different Android/iOS versions and devices. You can define a matrix of environments that will run in parallel, saving time and ensuring broader test coverage.

For example, you can configure matrix jobs to test across multiple Android API levels to ensure your app runs well on different versions of Android:

```yaml
android-test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      api-level:
        - "34"
        - "35"
  steps:
    - name: Checkout Repository
      uses: actions/checkout@v4
    - name: Setup and Install Flutter
      uses: subosito/flutter-action@v2
      with:
        channel: "stable"
        cache: true
    - name: Enable Android
      run: flutter config --enable-android
    - name: Install Flutter Dependencies
      run: flutter pub get
    - name: Run Flutter Integration Tests
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: ${{ matrix.api-level }}
        arch: x86_64
        profile: pixel_6_pro
        script: flutter test integration_test
    - name: Build Flutter App
      run: flutter build apk --debug
```

### Examples

Here are some examples from 2023-MarineConservationApp's project last year. They had a Flutter Package, Flutter Web App, and Android and iOS apps so they had quite a complex pipeline.

#### Example 1

```yaml
name: Continuous Integration

on:
  workflow_dispatch:
  pull_request:
  merge_group:

permissions:
  checks: write
  contents: write

jobs:
  test-lint-backend:
    name: Test and Lint Django Backend
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: "mca"
    services:
      postgres:
        image: postgres:latest
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: mca_test
        ports:
          - 5432:5432
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: Setup Environment Variables
        run: |
 touch .env
 echo ENVIRONMENT=ci >> .env
 echo DATABASE_USER=${DATABASE_USERNAME} >> .env
 echo DATABASE_PASSWORD=${DATABASE_PASSWORD} >> .env
 echo DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY} >> .env
 echo EMAIL_HOST_PASSWORD=${EMAIL_HOST_PASSWORD} >> .env
 cat .env
        env:
          DATABASE_USERNAME: ${{ secrets.DATABASE_USERNAME }}
          DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
          DJANGO_SECRET_KEY: ${{ secrets.DJANGO_SECRET_KEY }}
          EMAIL_HOST_PASSWORD: ${{ secrets.EMAIL_HOST_PASSWORD }}
      - name: Setup Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: 3.12
          cache: "pip"
      - name: Install Dependencies
        run: |
 python -m pip install --upgrade pip
 pip install -r requirements.txt
 pip install pylint
      - name: Run Python Linter (Pylint)
        run: pylint .
      - name: Run Python Tests for MCA Web
        run: python manage.py test mca_web.tests --no-input
      - name: Run Python Tests for MCA
        run: python manage.py test mca.tests --no-input

  test-lint-lib:
    name: Test and Lint Flutter Library
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: "mca-lib"
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: Setup and Install Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: "stable"
          cache: true
      - name: Print Flutter Version
        run: flutter --version
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Static Code Analysis
        run: flutter analyze
      - name: Run Flutter Unit Tests
        run: flutter test test

  test-lint-web:
    name: Test and Lint Web App
    runs-on: ubuntu-latest
    needs: test-lint-lib
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
      - name: Print Flutter Version
        run: flutter --version
      - name: Enable Web
        run: flutter config --enable-web
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Linter
        run: flutter analyze
      - name: Run Flutter Unit and Widget Tests
        run: flutter test test
      - name: Start Chromedriver for Integration Tests
        run: chromedriver --port=4444 &
      - name: Run Flutter Integration Tests
        run: flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart -d web-server --headless

  test-lint-app:
    name: Test and Lint iOS and Android App
    runs-on: ubuntu-latest
    needs: test-lint-lib
    defaults:
      run:
        working-directory: "mca-app"
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: Setup and Install Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: "stable"
          cache: true
      - name: Print Flutter Version
        run: flutter --version
      - name: Enable iOS
        run: flutter config --enable-ios
      - name: Enable Android
        run: flutter config --enable-android
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Linter
        run: flutter analyze
      - name: Run Flutter Unit and Widget Tests
        run: flutter test test

  validate-pr:
    name: Validate Pull Request
    runs-on: ubuntu-latest
    needs: [test-lint-backend, test-lint-lib, test-lint-web, test-lint-app]
    if: always()
    steps:
      - name: Check if all dependencies passed
        run: |
 if [[ "${{ needs.test-lint-backend.result }}" != "success" || \
 "${{ needs.test-lint-lib.result }}" != "success" || \
 "${{ needs.test-lint-web.result }}" != "success" || \
 "${{ needs.test-lint-app.result }}" != "success" ]]; then
 echo "One or more dependent jobs failed."
 exit 1
 else
 echo "All dependent jobs passed successfully."
 fi
        if: always()
```

#### Example 2

> [!CAUTION]
> This second example of MarineConservationApp's CI from last year did use up a significant proportion of the budget allocated to the organisation's GitHub Actions. This is only meant to serve as an example and should be tweaked if to be used this year.

```yaml
name: Continuous Integration - Full Suite

on:
  workflow_dispatch:

permissions:
  checks: write
  contents: write

jobs:
  test-lint-backend:
    name: Test and Lint Django Backend
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: "mca"
    services:
      postgres:
        image: postgres:latest
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: mca_test
        ports:
          - 5432:5432
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: Setup Environment Variables
        run: |
 touch .env
 echo ENVIRONMENT=ci >> .env
 echo DATABASE_USER=${DATABASE_USERNAME} >> .env
 echo DATABASE_PASSWORD=${DATABASE_PASSWORD} >> .env
 echo DJANGO_SECRET_KEY=${DJANGO_SECRET_KEY} >> .env
 echo EMAIL_HOST_PASSWORD=${EMAIL_HOST_PASSWORD} >> .env
 cat .env
        env:
          DATABASE_USERNAME: ${{ secrets.DATABASE_USERNAME }}
          DATABASE_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
          DJANGO_SECRET_KEY: ${{ secrets.DJANGO_SECRET_KEY }}
          EMAIL_HOST_PASSWORD: ${{ secrets.EMAIL_HOST_PASSWORD }}
      - name: Setup Python 3.12
        uses: actions/setup-python@v5
        with:
          python-version: 3.12
          cache: "pip"
      - name: Install Dependencies
        run: |
 python -m pip install --upgrade pip
 pip install -r requirements.txt
 pip install pylint
      - name: Run Python Linter (Pylint)
        run: pylint .
      - name: Run Python Tests for MCA Web
        run: python manage.py test mca_web.tests --no-input
      - name: Run Python Tests for MCA
        run: python manage.py test mca.tests --no-input

  test-lint-lib:
    name: Test and Lint Flutter Library
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: "mca-lib"
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v4
      - name: Setup and Install Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: "stable"
          cache: true
      - name: Print Flutter Version
        run: flutter --version
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Static Code Analysis
        run: flutter analyze
      - name: Run Flutter Tests
        run: flutter test test

  test-lint-web:
    name: Test and Lint Web App
    runs-on: ubuntu-latest
    needs: test-lint-lib
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
      - name: Print Flutter Version
        run: flutter --version
      - name: Enable Web
        run: flutter config --enable-web
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Linter
        run: flutter analyze
      - name: Run Flutter Unit and Widget Tests
        run: flutter test test
      - name: Start Chromedriver for Integration Tests
        run: chromedriver --port=4444 &
      - name: Run Flutter Integration Tests
        run: flutter drive --driver=test_driver/integration_test.dart --target=integration_test/app_test.dart -d web-server --headless
      - name: Build Web App
        run: flutter build web

  test-lint-ios-app:
    name: Test and Lint Flutter iOS App
    runs-on: macos-13
    needs: test-lint-lib
    defaults:
      run:
        working-directory: "mca-app"
    strategy:
      matrix:
        device:
          - "iPhone 15"
      fail-fast: true
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
      - name: Setup and Install Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: "stable"
          cache: true
      - name: Print Flutter Version
        run: flutter --version
      - name: Enable iOS
        run: flutter config --enable-ios
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Static Code Analysis
        run: flutter analyze
      - name: Run Flutter Unit and Widget Tests
        run: flutter test test
      - name: Run Flutter Integration Tests on ${{ matrix.device }}
        run: |
 xcrun simctl boot "${{ matrix.device }}"
 flutter test integration_test -d "${{ matrix.device }}"
      - name: Build Flutter App
        run: flutter build ios --no-codesign

  test-lint-android-app:
    name: Test and Lint Flutter Android App
    runs-on: ubuntu-latest
    needs: test-lint-lib
    defaults:
      run:
        working-directory: "mca-app"
    strategy:
      matrix:
        api-level:
          - "34"
        device:
          - "pixel_6_pro"
      fail-fast: true
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
      - name: Setup and Install Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: "stable"
          cache: true
      - name: Print Flutter Version
        run: flutter --version
      - name: Enable Android
        run: flutter config --enable-android
      - name: Run Flutter Doctor
        run: flutter doctor -v
      - name: Install Flutter Dependencies
        run: |
 flutter pub get
 flutter pub upgrade
      - name: Run Flutter Static Code Analysis
        run: flutter analyze
      - name: Run Flutter Unit and Widget Tests
        run: flutter test test
      - name: Run Flutter Integration Tests on ${{ matrix.device }} using API Level ${{ matrix.api-level }}
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: ${{ matrix.api-level }}
          arch: x86_64
          profile: ${{ matrix.device}}
          script: flutter test integration_test
          working-directory: ./mca-app
      - name: Build Flutter App
        run: flutter build apk --debug

  validate-pr:
    name: Validate Pull Request
    runs-on: ubuntu-latest
    needs:
      [
        test-lint-backend,
        test-lint-lib,
        test-lint-web,
        test-lint-ios-app,
        test-lint-android-app,
      ]
    if: always()
    steps:
      - name: Check if all dependencies passed
        run: |
 if [[ "${{ needs.test-lint-backend.result }}" != "success" || \
 "${{ needs.test-lint-lib.result }}" != "success" || \
 "${{ needs.test-lint-web.result }}" != "success" || \
 "${{ needs.test-lint-ios-app.result }}" != "success" || \
 "${{ needs.test-lint-android-app.result }}" != "success" ]]; then
 echo "One or more dependent jobs failed."
 exit 1
 else
 echo "All dependent jobs passed successfully."
 fi
        if: always()
```
