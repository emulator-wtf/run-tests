# emulator.wtf GitHub Action

Emulator.wtf is an Android cloud emulator laser-focused on performance to
deliver quick feedback to your PRs.

With this action you can easily run your Android instrumentation tests with
[emulator.wtf](https://emulator.wtf).

## Quick example

A really basic example building an app apk, test apk, running tests and then
finally collecting the results with the `mikepenz/action-junit-report@v2`
action:

```yaml
name: Run tests
on: push
jobs:
  run-tests:
    runs-on: ubuntu-20.04
  steps:
    - uses: actions/checkout@v2
    - name: Build app
      run: ./gradlew assembleDebug assembleAndroidTest
    - name: Run tests
      uses: emulator-wtf/run-tests@v0
      with:
        api-token: ${{ secrets.EW_API_TOKEN }}
        app: app/build/outputs/apk/debug/app-debug.apk
        test: app/build/outputs/apk/androidTest/app-debug-androidTest.apk
        outputs-dir: build/test-results
    - name: Publish test report
      uses: mikepenz/action-junit-report@v2
      if: always() # always run even if the tests fail
      with:
        report_paths: 'build/test-results/**/*.xml'
```

## Inputs

| Variable                   | Description                                                                                                                      |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| `version`                  | ew-cli version to use                                                                                                            |
| `api-token`                | API token for emulator.wtf. We recommend using a secret for this.                                                                |
| `app`                      | Path to application apk file                                                                                                     |
| `test`                     | Path to test apk file                                                                                                            |
| `library-test`             | Path to library test apk file                                                                                                    |
| `additional-apks`          | Additional apks to install, one per line                                                                                         |
| `file-cache-ttl`           | How long to cache test files for. Defaults to 1h.                                                                                |
| `file-cache`               | Whether to cache files between test runs. Defaults to true.                                                                      |
| `display-name`             | Display name for the test run                                                                                                    |
| `devices`                  | Device configurations to use, in the form of `model=X,version=Y` per line                                                        |
| `use-orchestrator`         | Whether to use the Android Test Orchestrator                                                                                     |
| `clear-package-data`       | Whether to clear app data between every test (requires `use-orchestrator`)                                                       |
| `num-flaky-test-attempts`  | Number of times to retry flaky tests. Defaults to 0.                                                                             |
| `flaky-test-repeat-mode`   | Whether to repeat only flaky tests (failed_only) or full shards (all) with flaky test attempts. Defaults to failed_only.         |
| `timeout`                  | Timeout for the test run, number with unit (`h`, `m` or `s`). Defaults to 15m.                                                   |
| `test-targets`             | Test targets to run, e.g. `class foo.bar.Baz` will only run tests in that class                                                  |
| `environment-variables`    | Environment variables to pass to AndroidJUnitRunner, one per line in the form of `key=value`                                     |
| `test-cache`               | Whether to cache test results between test runs. Defaults to true.                                                               |
| `side-effects`             | Whether the test has any side effects, like hitting external APIs. Prevents any test caching and retries.                        |
| `num-balanced-shards`      | Set to a number larger than 1 to split your tests into multiple shards based on test execution time to be executed in parallel   |
| `shard-target-runtime`     | Target a specific runtime (in minutes), this will spawn as many shards as needed to hit the target runtime.                      |
| `num-uniform-shards`       | Set to a number larger than 1 to randomly split your tests into multiple shards to be executed in parallel                       |
| `num-shards`               | Set to a number larger than 1 to split your tests into multiple shards based on test counts to be executed in parallel           |
| `record-video`             | Set to true to record a video of the test run. Defaults to false.                                                                |
| `with-coverage`            | Set to true to collect coverage files and save them to `outputs-dir`                                                             |
| `directories-to-pull`      | Directories to pull from device and store in `outputs-dir`, one per line                                                         |
| `outputs`                  | Comma-separated list to specify what to download to output-dir. Defaults to `merged_results_xml,coverage,pulled_dirs`.           |
| `outputs-dir`              | Location to store test outputs in                                                                                                |
| `proxy-host`               | Configure a proxy host to use for all requests when making emulator.wtf API calls.                                               |
| `proxy-port`               | Configure a proxy port to use for all requests when making emulator.wtf API calls.                                               |
| `proxy-user`               | Set the proxy user to use for authentication.                                                                                    |
| `proxy-password`           | Set the proxy password to use for authentication.                                                                                |
| `async`                    | Run the test asynchronously, without waiting for the results. Useful if you're using the GitHub integration with check statuses. |

## Common examples

### Run tests with multiple device profiles

By default emulator.wtf runs tests on a Pixel2-like emulator with API 27
(Android 8.1). If you want to run on a different version or device profile you
can use the `devices` input to do so:

```yaml
    - name: Run tests
      uses: emulator-wtf/run-tests@v0
      with:
        api-token: ${{ secrets.EW_API_TOKEN }}
        app: app/build/outputs/apk/debug/app-debug.apk
        test: app/build/outputs/apk/androidTest/app-debug-androidTest.apk
        devices: |
          model=NexusLowRes,version=23
          model=Pixel2,version=27
```

### Run tests with orchestrator while clearing package data

You can use Android Test Orchestrator to run the tests - this will create a new
app VM from scratch for each test. Slower to run, but will ensure no static
state leakage between tests. Add the optional `clear-package-data` flag to clear
app persisted state between each run. Read more about orchestrator
[here](https://developer.android.com/training/testing/junit-runner#using-android-test-orchestrator).

```yaml
    - name: Run tests
      uses: emulator-wtf/run-tests@v0
      with:
        api-token: ${{ secrets.EW_API_TOKEN }}
        app: app/build/outputs/apk/debug/app-debug.apk
        test: app/build/outputs/apk/androidTest/app-debug-androidTest.apk
        use-orchestrator: true
        clear-package-data: true
```

### Grab coverage data

Use the `with-coverage` flag to capture test run coverage data and store the
results (one or more `.exec` or `.ec` files) in the path specified by
`outputs-dir`:

```yaml
    - name: Run tests
      uses: emulator-wtf/run-tests@v0
      with:
        api-token: ${{ secrets.EW_API_TOKEN }}
        app: app/build/outputs/apk/debug/app-debug.apk
        test: app/build/outputs/apk/androidTest/app-debug-androidTest.apk
        with-coverage: true
        outputs-dir: build/test-results
```


### Run tests with shards

The following example runs tests in parallel using 3 separate shards and stores
the outputs from each shard in a separate folder under `build/test-results`:

```yaml
    - name: Run tests
      uses: emulator-wtf/run-tests@v0
      with:
        api-token: ${{ secrets.EW_API_TOKEN }}
        app: app/build/outputs/apk/debug/app-debug.apk
        test: app/build/outputs/apk/androidTest/app-debug-androidTest.apk
        num-shards: 3
        outputs-dir: build/test-results
```
