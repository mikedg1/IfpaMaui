# iOS Local Development Setup & Customization Guide

This document describes helpful configuration changes to build and run the IFPA Companion .NET MAUI application as a contributor, along with the rationale for each modification.

---

## Overview of Local Changes

When building the iOS target locally, the production settings in the repository (which reference CI signing certificates, production bundle identifiers, and build-time secret injection) must be adjusted for your local Apple Developer account and development environment.

The key areas modified for local iOS development are:
1. **Application Bundle Identifier**
2. **Code Signing Key and Provisioning Profile**
3. **Disabling Native Extension Pre-builds (Widget Extensions)**
4. **IFPA API Key Configuration**

There may be ways to avoid some or all of these, if so, please let me know.

---

## Detailed Breakdown of Changes

### 1. Application Bundle Identifier
* **Files Modified**:
  - `src/IfpaMaui/IfpaMaui.csproj.user`
* **Changes**:
  Override `ApplicationId` in your local `IfpaMaui.csproj.user` to a custom bundle id:
  ```xml
  <PropertyGroup>
      <ApplicationId>com.example.ifpa</ApplicationId> <!-- your personal developer bundle ID -->
  </PropertyGroup>
  ```
* **Rationale**:
  Apple requires bundle identifiers to be unique per developer account. Deploying to a local physical iOS device or simulator requires a bundle ID registered under your team's Apple Developer Account (or automatically managed by Xcode's personal team provisioning profile).

---

### 2. Code Signing Key & Provisioning Profile
* **Files Modified**:
  - `src/IfpaMaui/IfpaMaui.csproj.user`
* **Changes**:
  Override `CodesignKey` and `CodesignProvision` in your local `IfpaMaui.csproj.user`:
  ```xml
  <PropertyGroup Condition="'$(TargetFramework)'=='net10.0-ios' and '$(Configuration)' == 'Debug'">
      <CodesignKey>Apple Development: Developer Name (EEEBSAJETH)</CodesignKey>
      <CodesignProvision>iOS Team Provisioning Profile: com.example.ifpa</CodesignProvision>
  </PropertyGroup>
  ```
* **Rationale**:
  The repository is configured by default for CI/automated release signing. For local builds on macOS targeting a physical iOS device, `CodesignKey` and `CodesignProvision` must match the signing identity and provisioning profile installed in your local Keychain and Xcode profile cache.

---

### 3. Disabling Native App Extensions (`AdditionalAppExtensions`)
* **Files Modified**:
  - `src/IfpaMaui/IfpaMaui.csproj.user`
* **Changes**:
  Clear `AdditionalAppExtensions` in your local `IfpaMaui.csproj.user`by adding the following:
  ```xml
  <ItemGroup>
      <AdditionalAppExtensions Remove="@(AdditionalAppExtensions)" />
  </ItemGroup>
  ```
* **Rationale**:
  The `AdditionalAppExtensions` element tells MSBuild/MAUI to embed the pre-compiled Swift WidgetKit extension (`RankWidgetExtension`) into the main app bundle. 
  
  During local development:
  - The native widget binaries under `DerivedData` might not be pre-built prior to running `dotnet build`, which triggers MSBuild error: `The source '.../RankWidgetExtension.appex' does not exist`.
  - The embedded extension bundle ID (`com.edgiardina.ifpa.rankwidget`) and entitlements require matching App Group and Provisioning Setup.
  - Removing `AdditionalAppExtensions` in `IfpaMaui.csproj.user` allows rapid inner-loop development of the main MAUI iOS app without needing full native widget pre-builds or signing setup.

---

### 4. Local IFPA API Key Setup
* **Files Modified**:
  - `src/IfpaMaui/appsettings.json`
* **Changes**:
  Added the local developer IFPA API Key (`IfpaApiKey`) to `appsettings.json`.
* **Rationale**:
  The IFPA API (`api.ifpapinball.com`) requires an API key for queries. In publication pipelines, secrets are injected during CI/CD build actions. For local development, an API key must be specified locally for network requests to return data.

---

## Best Practices for Managing Local Changes in Git

To prevent committing personal signing credentials, developer bundle IDs, or API keys back to the remote repository:

1. **Use `IfpaMaui.csproj.user` for Local Overrides (Recommended)**:
   Instead of modifying `src/IfpaMaui/IfpaMaui.csproj` directly, create a local override file at `src/IfpaMaui/IfpaMaui.csproj.user`. MSBuild automatically imports this file after evaluating `IfpaMaui.csproj`, allowing your personal developer properties to override the defaults. 
   
   Since `*.user` is matched by `.gitignore`, this file will remain ignored by Git and will never appear in `git status` or pull requests.

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <Project>
       <PropertyGroup>
           <ApplicationId>com.mikedg.ifpa</ApplicationId>
       </PropertyGroup>

       <PropertyGroup Condition="'$(TargetFramework)'=='net10.0-ios' and '$(Configuration)' == 'Debug'">
           <CodesignKey>Apple Development: Michael DiGovanni (25CBSAJETH)</CodesignKey>
           <CodesignProvision>iOS Team Provisioning Profile: com.mikedg.ifpa</CodesignProvision>
       </PropertyGroup>

       <!-- Removes WidgetKit extension embedding for local builds -->
       <ItemGroup>
           <AdditionalAppExtensions Remove="@(AdditionalAppExtensions)" />
       </ItemGroup>
   </Project>
   ```

2. **Avoid Staging Local Config Changes**:
   Do not add `appsettings.json` with active API keys or `IfpaMaui.csproj` with personal certificate names to commits meant for pull requests.

3. **Temporarily Ignore File Changes (Git Assume Unchanged)**:
   If you want Git to ignore local edits to tracked files while working on features:
   ```bash
   git update-index --assume-unchanged src/IfpaMaui/appsettings.json
   ```
   To revert back:
   ```bash
   git update-index --no-assume-unchanged src/IfpaMaui/appsettings.json
   ```

4. **Restoring Before Pull Request Submission**:
   Before creating a PR, restore configuration files:
   ```bash
   git checkout -- src/IfpaMaui/IfpaMaui.csproj src/IfpaMaui/appsettings.json src/IfpaMaui/NativeIFPA/
   ```

