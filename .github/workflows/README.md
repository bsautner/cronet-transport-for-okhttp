# GitHub Actions Workflows

This directory contains automated workflows for the Cronet Transport for OkHttp project.

## android-build.yml

This workflow builds the Android library artifact and runs tests.

### Trigger Events

The workflow runs on:
- **Push**: When code is pushed to `main` or `master` branches
- **Pull Request**: When a PR is opened or updated targeting `main` or `master` branches
- **Manual**: Can be triggered manually from the Actions tab using `workflow_dispatch`

### Build Environment

The workflow sets up:
- **Ubuntu Latest**: Linux environment for building
- **JDK 11**: Java Development Kit (Temurin distribution)
- **Android SDK**: Platform 32
- **Android NDK**: Version 21.4.7075529
- **Bazel**: Using bazelisk with caching enabled

### Build Steps

1. **Checkout code**: Clones the repository
2. **Setup Java**: Installs JDK 11
3. **Setup Android SDK**: Installs Android SDK and required components
4. **Install Android components**: Installs platform-tools, platform 32, build-tools 30.0.2, and NDK 21
5. **Setup Bazel**: Installs Bazel with caching for faster builds
6. **Configure environment**: Sets `ANDROID_HOME` and `ANDROID_NDK_HOME` environment variables
7. **Build library**: Executes `bazel build //:okhttp_cronet_transport`
8. **List artifacts**: Shows what files were built for visibility
9. **Run tests**: Executes `bazel test //javatests/...` (continues on error)
10. **Archive artifacts**: Uploads build outputs and test results

### Artifacts

The workflow produces two sets of artifacts:

#### Library Artifacts
- **Name**: `library-artifacts`
- **Contents**: All `.aar` and `.jar` files from `bazel-bin/`, plus any files matching `bazel-bin/**/okhttp_cronet_transport*`
- **Retention**: 30 days
- **Download from**: Actions tab → Select workflow run → Artifacts section

#### Test Results
- **Name**: `test-results`
- **Contents**: Test logs from `bazel-testlogs/`
- **Retention**: 7 days
- **Download from**: Actions tab → Select workflow run → Artifacts section

### Build Commands

The main build command used (builds the primary library target):
```bash
bazel build //:okhttp_cronet_transport
```

Note: This builds the main library target specifically. To build the entire repository as mentioned in CONTRIBUTING.md, you would use:
```bash
bazel build //...
```

The test command used (runs all tests, continues on error):
```bash
bazel test //javatests/...
```

**Important**: Tests are configured with `continue-on-error: true`, meaning the workflow will succeed and produce artifacts even if some tests fail. Check the test-results artifact to see which tests passed or failed.

### Environment Variables

- `ANDROID_HOME`: Set to `$ANDROID_SDK_ROOT` (provided by android-actions/setup-android)
- `ANDROID_NDK_HOME`: Set to `$ANDROID_SDK_ROOT/ndk/21.4.7075529`

### Caching

The workflow uses several caching mechanisms to speed up builds:
- **Bazelisk cache**: Caches the Bazel installation
- **Disk cache**: Caches Bazel build outputs
- **Repository cache**: Caches external dependencies

### Permissions

The workflow requires:
- `contents: read` - To checkout the code
- `actions: read` - To access workflow artifacts

### Troubleshooting

If the workflow fails:

1. **Check the workflow run**: Go to the Actions tab and select the failed run
2. **Review logs**: Click on the failed step to see detailed logs
3. **Download artifacts**: Even failed builds may produce partial artifacts
4. **Test results**: Download test-results artifact to see which tests failed

Common issues:
- **SDK/NDK not found**: The workflow automatically installs these, but if there are issues, check the "Install Android SDK components" step
- **Build failures**: Check the "Build Android library" step logs
- **Test failures**: Tests are set to continue on error, so build artifacts are still produced

### Local Testing

To test the workflow locally before pushing:
1. Ensure you have the required tools installed (see [CONTRIBUTING.md](../../CONTRIBUTING.md))
2. Run the build command: `bazel build //:okhttp_cronet_transport`
3. Run tests: `bazel test //javatests/...`

### Modifying the Workflow

If you need to modify the workflow:
1. Edit `.github/workflows/android-build.yml`
2. Test your changes locally if possible
3. Validate YAML syntax: `python -c "import yaml; yaml.safe_load(open('.github/workflows/android-build.yml'))"`
4. Create a PR with your changes
5. The workflow will run on your PR, allowing you to verify the changes work correctly
