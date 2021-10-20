# emulator.wtf GitHub Action

Emulator.wtf is an Android cloud emulator laser-focused on performance to
deliver quick feedback to your PRs.

With this action you can easily run your Android instrumentation tests with
emulator.wtf.

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
      uses: emulator-wtf/run-tests@master
      with:
        api-token: ${{ secrets.EW_API_TOKEN }}
        app-apk: app/build/outputs/apk/debug/app-debug.apk
        test-apk: app/build/outputs/apk/androidTest/app-debug-androidTest.apk
        outputs-dir: build/test-results
    - name: Publish test report
      uses: mikepenz/action-junit-report@v2
      if: always() # always run even if the tests fail
      with:
        report_patrhs: 'build/test-results/**/*.xml'
```

## Inputs

| variable | description |
| -------- | ----------- |
| version | ew-cli version to use |
| api-token | Api token for emulator.wtf. We recommend using a secret for this. |
| app-apk | Path to application apk file |
| test-apk | Path to test apk file |
| outputs-dir | Location to store test outputs in |
| devices | Device configurations to use, in the form of `model=X,version=Y` per line |
| use-orchestrator | Whether to use the Android Test Orchestrator |
| clear-package-data | Whether to clear app data between every test (requires `use-orchestrator`) |
| with-coverage | Set to true to collect coverage files and save them to `outputs-dir` |
| additional-apks | Additional apks to install, one per line |
| environment-variables | Environment variables to pass to AndroidJUnitRunner, one per line in the form of `key=value` |
| num-uniform-shards | Set to a number larger than 1 to randomly split your tests into multiple shards to be executed in parallel |
| directories-to-pull | Directories to pull from device and store in `outputs-dir`, one per line |
