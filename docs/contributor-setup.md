# Contributor Setup & Development Guide

This guide outlines the essential prerequisites and setup steps required to build and run the IFPA Companion .NET MAUI application on macOS for local development.

---

## Rationale

- **.NET SDK & MAUI Workload**: The application is built on modern .NET (.NET 10) and .NET MAUI. The base .NET SDK provides the CLI and runtime, while the `maui` workload installs the necessary platform SDKs (Android, iOS, MacCatalyst), toolchains, and MSBuild targets required to compile mobile binaries.
- **Java Development Kit (JDK 21) for Android**: The Android build pipeline (using Android SDK build tools, AAPT2, and Java/Kotlin interop) relies on a compatible JDK. If the system defaults to an unsupported or older Java version, the build will fail with Java compilation or version mismatch errors.
- **Verification via Test Builds**: Building for specific target frameworks (`net10.0-android` and `net10.0-ios`) ensures all platform dependencies, workloads, and toolchains are installed and functioning properly before starting development.

---

## 1. Install Build Toolkits

Install the latest .NET SDK and the required MAUI workloads.

### Install .NET SDK
On macOS with [Homebrew](https://brew.sh/):
```bash
brew install --cask dotnet-sdk
```

### Install .NET MAUI Workloads
Install the MAUI workload to obtain Android and iOS targeting packs:
```bash
sudo dotnet workload install maui
```

---

## 2. Configure Java for Android Builds (If Needed)

Android compilation requires a compatible Java environment (such as OpenJDK 21). If your Android build complains about Java versions or missing Java tools, set `JAVA_HOME` in your environment:

```bash
export JAVA_HOME="/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home"
```

---

## 3. Verify with Test Builds

Run a test build for your target platform to ensure the environment is configured correctly.

### Android
```bash
dotnet build src/IfpaMaui/IfpaMaui.csproj -c Release -f net10.0-android
```

### iOS
```bash
dotnet build src/IfpaMaui/IfpaMaui.csproj -c Release -f net10.0-ios
```

---

## Next Steps & Platform-Specific Details

- **iOS Local Development & Signing**: For local iOS simulator and device builds, bundle identifier customization, code signing, and native extension configurations, refer to the [iOS Contributor Setup Guide](ios--contributor-development-setup.md).
- **Running Tests**: You can verify unit tests locally with:
  ```bash
  dotnet test tests/IfpaMaui.Tests/IfpaMaui.Tests.csproj
  ```
