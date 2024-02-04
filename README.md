[![Build and Test](https://github.com/bmuschko/setup-gradle-public-build-scan/actions/workflows/build-test.yml/badge.svg)](https://github.com/bmuschko/setup-gradle-public-build-scan/actions/workflows/build-test.yml)

# setup-gradle-public-build-scan

A GitHub Action that sets up a Gradle initialization script for creating a build scan on [scans.gradle.com](https://docs.gradle.com/enterprise/gradle-plugin/#connecting_to_scans_gradle_com). A build scan on scans.gradle.com is commonly-referred to as a public build scan, as anyone with access to the build scan URL will be able to view the data. A public build scan is useful for capturing Gradle build metrics for projects developed in the open without having to protect sensitive data. By applying this action, you do not have to configure the [Develocity Gradle plugin](https://docs.gradle.com/enterprise/gradle-plugin/) in your Gradle build code anymore.

> [!NOTE]
> This action _cannot_ be configured to send build metrics to a company-internal Develocity server. Refer to the `actions/setup-gradle` to achieve that.

The configuration generated by this GitHub Action auto-accepts the terms of service and will send metrics for every build invocation. Furthermore, the initialization script applies the [Common Custom User Data Gradle Plugin](https://github.com/gradle/common-custom-user-data-gradle-plugin) to enhance the published build scan by adding a set of common tags and links.

> [!IMPORTANT]
> This action requires the use of Gradle 5.0 or higher, the Develocity Gradle plugin version 3.0 or higher, and the Common Custom User Data Gradle plugin version 1.0 or higher.

## Using the Action

It's recommended to use the action as a single setup step. The following example shows its usage.

```yaml
name: Build Gradle project
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Set up Gradle
      uses: gradle/actions/setup-gradle@v3
      with:
        gradle-version: current

    - name: Set up build scan
      uses: bmuschko/setup-gradle-public-build-scan@v1

    - name: Execute Gradle build
      run: gradle integ-test
```

Every Gradle build invocation will render the build scan URL to the terminal as part of its output.

## Action Inputs and Outputs

The action defines [inputs](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#inputs) and [outputs](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions#outputs-for-docker-container-and-javascript-actions). The inputs let you drive the runtime behavior of the action. The outputs retrieve values populated after executing the action.

### Inputs

|Parameter|Required|Default Value|Description|
|:--:|:--:|:--:|:--|
|`develocity-gradle-plugin-version`|`true`|`3.16.2`|The Develocity Gradle plugin version that provides build scan functionality.|
|`common-user-data-plugin-version`|`true`|`1.12.1`|The Gradle plugin version that provides common user data with build scan.|
|`tags`|`false`|`null`|Tags to send with build scan defined in the format of a JSON array, e.g. `["integ-test", "ubuntu-latest"]`.|
|`links`|`false`|`null`|Links to send with build scan defines in the format of a JSON map, e.g. `{"Developer": "https://github.com/bmuschko/"]}`.|

### Outputs

|Parameter|Description|
|:--:|:--:|
|`init-script-path`|The path to the init script that sets up the build scan functionality.|

### Example

The following example shows the use of both inputs and outputs. Here, the default version of Develocity Gradle plugin and the Common Custom User Data Gradle plugin has been changed. Furthermore, extra tags and links have been provided. The workflow adds a step that renders the path to the initialization script after it has been created.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Set up build scan
      id: setup-build-scan
      uses: bmuschko/setup-gradle-public-build-scan@v1
      with:
        develocity-gradle-plugin-version: '3.16.1'
        common-user-data-plugin-version: '1.12.1'
        tags: '["integ-test", "ubuntu-latest"]'
        links: '{"Developer": "https://github.com/bmuschko"}'

    - name: Print initialization script path
      env:
        INIT_SCRIPT_PATH: ${{ steps.setup-build-scan.outputs.init-script-path }}
      run: |
          echo "Intialization script path: ${INIT_SCRIPT_PATH}"
      shell: bash
```