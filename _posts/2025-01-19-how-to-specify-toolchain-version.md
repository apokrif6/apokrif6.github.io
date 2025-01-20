---
title: How to specify toolchain version in Unreal Engine
---
<h3>What is toolchain</h3>
A toolchain refers to the collection of software tools used to compile, link, and package a game or application for a specific platform. It typically includes compilers, linkers, libraries, and other utilities needed to build and deploy projects

<h3>Problem</h3>
The default toolchain used by Unreal Engine for Windows development is `Microsoft Visual C++ (MSVC)`.
This toolchain is automatically configured when you set up Visual Studio using Unreal Engine’s official [documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine).

![IncludedToolchain](https://apokrif6.github.io/assets/images/specify_toolchain_version/example_included_toolchain.png)

For most projects, this setup works seamlessly and ensures compatibility between the engine and the build process.

However, issues can arise in specific cases where:
- The default toolchain version is incompatible with the Unreal Engine version you're using. 
- The latest compiler version introduces bugs or behavior changes that affect the engine or project.
- Unreal Engine explicitly bans certain toolchain versions due to known issues.


One of those issues is communicated by `UnrealBuildTool` when you try to compile the project:
```
ERROR: UnrealBuildTool has banned the MSVC 14.32.31326-14.32.31328 toolchains due to compiler issues. Please install a different toolchain from the Visual Studio installer
```

To manage this, firstly you should install specific toolchain version. Do it with `Visual Studio Installer`:
- Find your Visual Studio
- Click `Modify`
- Click `Individual components`
- Type `MSVC` in the search bar
- Find needed version and mark it
- Click `Modify`

![IndividualComponent](https://apokrif6.github.io/assets/images/specify_toolchain_version/msvc_individual_component.png)

Now you have installed specific version of the toolchain. But how to specify it to work with `UnrealBuildTool`?
You should use [BuildConfiguration](https://dev.epicgames.com/documentation/en-us/unreal-engine/build-configuration-for-unreal-engine) files.
There are a lot of options available, but we are interested in `CompilerVersion`.
It is used to specify the toolchain to use for building the project.
```
$ CompilerVersion : The specific compiler version to use. This may be a specific version number (for example, "14.13.26128"), the string "Latest" to select the newest available version, or the string "Preview" to select the newest available preview version.
By default, and if it is available, we use the toolchain version indicated by WindowsPlatform.DefaultToolChainVersion (otherwise, we use the latest version).
```

So, to specify the toolchain version, you should add this line to your `BuildConfiguration.xml` file:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">
<WindowsPlatform>
        <CompilerVersion>
            14.38.33130 <!-- Specifies the exact toolchain version to use -->
        </CompilerVersion>
    </WindowsPlatform>
</Configuration> 
```

<h3>Conclusion</h3>
Now, when you try to compile the project, `UnrealBuildTool` will use the specified toolchain version. No warnings or errors will be thrown, and the project will compile successfully.

The `BuildConfiguration` file is more than just a way to specify toolchain versions—it's a versatile tool for tailoring your build process to meet the specific needs of your project. You can use it to optimize build performance, customize platform-specific settings, or troubleshoot issues by adjusting advanced build parameters.
Here are a few examples of what else you can configure with BuildConfiguration:
- Enable or Disable Unity Builds: Speed up build times for large projects by combining source files into fewer compilation units.
- Debugging Enhancements: Adjust settings to improve symbol generation or logging for debugging builds.
- Toolchain Overrides: Use alternate compilers, such as Clang for Windows, or specify preview versions for testing.