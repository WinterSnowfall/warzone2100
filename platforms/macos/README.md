# Building Warzone 2100 for macOS

## Prerequisites:

> For convenience, you will probably want either [Homebrew](https://brew.sh) or [Macports](https://www.macports.org/install.php) installed for setting up certain prerequisites. If you don't have either yet, **Homebrew** is recommended.

1. **macOS 13+**
    - While building may work on prior versions of macOS, CI only tests on macOS 13+.

2. **Xcode 14+** (tested w/: Xcode 14.x)
    - If you do not have Xcode 14.3.1+ you can get it for free at the [Mac App Store](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) or [Apple's website](http://developer.apple.com/technology/xcode.html).
    - Earlier versions of Xcode may work, but are not tested by CI.

3. **CMake 3.24+** (required to generate the Xcode project)
    - If you do not have CMake 3.24+, you can [download the latest stable version for free at CMake.org](https://cmake.org/download/#latest).

4. **Autoconf** (required to compile some vcpkg ports)
    - If you have [Homebrew](https://brew.sh) installed, you can use the following command in Terminal:
        ```shell
        brew install automake
        ```
        > See https://github.com/Homebrew/homebrew-core/issues/21125 for why the `automake` package must be installed instead of `autoconf`

5. **Gettext** (required to compile the translations and language files)
    - If you have [Homebrew](https://brew.sh) installed, you can use the following command in Terminal:
        ```shell
        brew install gettext
        ```
        > The build scripts work perfectly with the default (keg-only) Homebrew install of gettext.
    - If you have [Macports](https://www.macports.org/install.php) installed, you can use the following command in Terminal:
        ```shell
        sudo port install gettext
        ```

6. **Asciidoctor** (required to build the documentation / help files)
    - If you have [Homebrew](https://brew.sh) installed, you can use the following command in Terminal:
        ```shell
        brew install asciidoctor
        ```
    - If you have [Macports](https://www.macports.org/install.php) installed, you can use the following command in Terminal:
        ```shell
        sudo port install asciidoctor
        ```
    - Or, via `gem install`:
        ```shell
        gem install asciidoctor --no-document
        ```
        > Depending on system configuration, `sudo gem install` may be required.

## Tips for Compiling on earlier macOS / Xcode:

> [!NOTE]
> Since Xcode supports targeting older versions of macOS as a minimum deployment target, we recommend building with versions of macOS + Xcode that match the recommendations above (or newer).

While CI does not test these combinations, WZ's CMake buildsystem previously compiled with:
- Xcode 11+ on macOS 10.15
- Xcode 8 / 9 / 10 on macOS 10.12-10.14

> However, this is not tested - just a guideline if you want to try something unsupported.

## Setup & Configuration:

Generating the macOS port's build environment & configuration requires the following steps in **Terminal**:

### 1. Create a build directory

The recommended place for the build directory is outside of the Git repo / source code.

For example,
- if you wanted to create a build directory one level up from the Git repo / source code
- and you cloned the Git repo to `~/src/warzone2100`

you could use the following commands:

```shell
cd ~/src/
mkdir wz_build
```

### 2. Run _configure_mac.cmake_ from the build directory

1. `cd` into the build directory you created in the step above
   ```shell
   cd wz_build
   ```

2. Run the `configure_mac.cmake` script:
   ```shell
   cmake -P "../warzone2100/configure_mac.cmake"
   ```
   Where the `../warzone2100/configure_mac.cmake` path should be modified to point to `configure_mac.cmake` inside the Git repo / source code directory.

The `configure_mac.cmake` script will automatically:
   - Download + extract the Vulkan SDK _(on macOS 10.14+ only - required for Vulkan / Metal support)_
   - Download + build [vcpkg](https://github.com/microsoft/vcpkg)
   - Build required dependencies
   - Run CMake to generate the Xcode project


## Building:

The macOS port is built using the Xcode project generated by CMake. If following the instructions above, this will be located in your build directory:
`<path_to_build_directory>/warzone2100.xcodeproj`

**To build the game from the command-line:**

1. `cd` into your build directory that contains the generated `warzone2100.xcodeproj`

2. Run the following command:
    ```shell
    xcodebuild -project warzone2100.xcodeproj -target warzone2100 -configuration Release -destination "platform=macOS"
    ```

You can also simply open the project in Xcode, and build the `warzone2100` scheme. (You may want to tweak it to build in a Release configuration - the default for that scheme in "Run" mode is Debug, for easier development.)


## Deployment:

The macOS port produces a 64-bit [self-contained application bundle](https://developer.apple.com/library/content/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW13) that requires **macOS 10.9+**\* to run.

> \* See _Setup & Configuration_ for how to set the minimum deployment target.

If built on macOS 10.14+, the application will support both the OpenGL and Vulkan (Metal) backends on macOS.

## Additional Information:

The macOS port supports [sandboxing](https://developer.apple.com/library/content/documentation/Security/Conceptual/AppSandboxDesignGuide/AboutAppSandbox/AboutAppSandbox.html), but this requires [code-signing](https://developer.apple.com/library/content/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html) to be configured in the Xcode project.
(By default, code-signing is disabled in the Xcode project.)