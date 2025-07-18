# OpenRV Build Instructions

Written instructions for building [OpenRV](https://github.com/AcademySoftwareFoundation/OpenRV), for easy copy/pasting. Will try to keep it updated as time passes and build commands/instructions change. The instructions are more or less taken from the official documentation, but may be altered slightly because of pitfalls I ran into or because I preferred to install certain utilities in a slightly different order. But there is nothing really special going on here, just following the instructions in the docs.

- Source code repository: https://github.com/AcademySoftwareFoundation/OpenRV
- Documentation: https://aswf-openrv.readthedocs.io/en/latest/build_system/config_macos.html

## Table of Contents

1. [Allow Terminal to update or delete other applications](#step_1)
2. [Install Homebrew](#step_2)
3. [Install tools and build dependencies](#step_3)
4. [Install CMake](#step_4)
5. [Install Qt](#step_5)
6. [Clone the OpenRV git repository](#step_6)
7. [Run the build commands](#step_7)
8. [Miscellaneous/Optional](#step_8)
	- [Non-free codecs](#non_free_codecs)
	- [Building OpenRV after the first time](#rebuilding)


# MacOS

## Step 1. Allow Terminal to update or delete other applications <a name="step_1"></a>

1. Go to MacOS `System Settings -> Privacy & Security -> App Management` and allow `Terminal` to update and delete other applications. Terminal is in `Applications -> Utilities -> Terminal`

## Step 2. Install Homebrew <a name="step_2"></a>

If you follow the steps in this order, it will also install XCode Command Line Tools, which skips the need to install the App Store version like the official OpenRV docs specify. The full XCode IDE from the app store can be over 30GBs in my experience, which is storage I am personally trying to save.

1. Go to the homebrew website: https://brew.sh/
2. Copy and paste their install command into Terminal, which runs the install script. It will require the `sudo` password when installing. When typing in the Terminal it is normal for characters not to appear when typing a password
3. Open a new Terminal window after installing Homebrew
4. Check the install worked with `brew --version`

## Step 3. Install tools and build dependencies <a name="step_3"></a>

In general, run `brew update` before installing new Homebrew packages.

Copy the command directly from OpenRV docs: https://aswf-openrv.readthedocs.io/en/latest/build_system/config_macos.html#install-tools-and-build-dependencies

Command: `brew install ninja readline sqlite3 xz zlib tcl-tk@8 autoconf automake libtool python@3.11 yasm clang-format black meson nasm pkg-config glew`

You can ignore the XCode command stuff since we installed it a different way.

## Step 4. Install CMake <a name="step_4"></a>

Homebrew installs CMake major version 4 and we need an earlier version to build OpenRV. The official docs recommend to download the image directly and adding `cmake` to `PATH` manually, but I prefer to install old versions via Homebrew still, even though it's a few more commands to run than normal. The choice is yours, but here is what I did.

1. Create new custom tap (you can choose any name) for archival Homebrew: `brew tap-new --no-git my-tap/homebrew-archive`
2. Clone the full Homebrew repo. This is quite large (over 1GB), but we will delete it later: `brew tap --force homebrew/core`
3. Extract an OpenRV compatible version of CMake (currently `v3.31.6`) that is in the Homebrew archives: `brew extract --version 3.31.6 cmake my-tap/homebrew-archive`
4. Install the newly extracted package: `brew install my-tap/homebrew-archive/cmake@3.31.6`
5. From a new terminal window, test out the cmake command now works and is the right version: `cmake --version`
6. If everything is good, remove the full Homebrew archive: `brew untap homebrew/core`
7. Turn off Homebrew developer mode: `brew developer off`

## Step 5. Install Qt <a name="step_5"></a>

First head over to the Qt website to download the installer: https://www.qt.io/download-qt-installer-oss

Unfortunately you have to create an account with them to use the online installer. Uncheck the preset options and choose Custom Installation when you get to that page. You may get an error about the XCode path not found, you can just ignore it. These are the minimal libraries need to keep the installation size small:


- Qt 6.5.3
	- macOS
	- Sources
	- Additional Libraries
		- Qt Multimedia
		- Qt Positioning
		- Qt WebChannel
		- Qt WebEngine
- Build Tools
	- CMake 3.30.5
	- Ninja 1.12.1
 
The CMake and Ninja versions aren't as crucial as long as the major version matches. The Qt version is important according to the OpenRV docs.

## Step 6. Clone the OpenRV git repository <a name="step_6"></a>

You will need the source code to build the project. First create a directory where you will store the project. 

For example: `mkdir -p ~/Documents/open_source`

Then change the active directory to the one you just created: `cd ~/Documents/open_source`

Finally clone the OpenRV repo with git: `git clone --recursive https://github.com/AcademySoftwareFoundation/OpenRV.git`

 ## Step 7. Run the build commands <a name="step_7"></a>

 First make sure you are inside the OpenRV repo when you are running the commands. Use the `pwd` command to check the current path. If you followed the above steps to clone the git repo, you can enter the OpenRV directory by running: `cd ~/Documents/open_source/OpenRV`

 Now you can run the build commands. The first time you build the project, it may take a long time to compile, since OpenRV is a complex piece of software.

 1. Run command: `source rvcmds.sh`
 2. Run command: `rvsetup`
 3. Bootstrap the project if you are running the build for the first time: `python3 -m pip install -r requirements.txt`
 4. Configure: `rvcfg`
 5. Build: `rvbuild`

After a successful build, the OpenRV executable can be found inside the project directory under `_build/stage/app/RV.app/Contents/MacOS/RV`

You can drag the `RV.app` file into the Applications directory on your Mac if you prefer.

## Step 8. Miscellaneous/Optional <a name="step_8"></a>

### Non-free codecs <a name="non_free_codecs"></a>

You can compile OpenRV with non-free audio and video codecs such as `AAC`, `HEVC`, etc. Please see the following `Legal Notice` copied from the OpenRV docs!

**Legal Notice: Non-free FFmpeg codecs are disabled by default. Please check with your legal department whether you have the proper licenses and rights to use these codecs. ASWF is not responsible for any unlicensed use of these codecs.**

This applies if you plan on distributing binaries with non-free codecs on the internet, i.e. providing pre-compiled builds for other people. Please do your research on these legal matters before distributing.

The instructions in the docs: https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html#how-to-enable-non-free-ffmpeg-codecs

To build with non-free codecs, add them as arguments to the `rvcfg` command. For example, to build with `aac` and `hevc` support: 

`rvcfg -DRV_FFMPEG_NON_FREE_DECODERS_TO_ENABLE="aac;hevc" -DRV_FFMPEG_NON_FREE_ENCODERS_TO_ENABLE="aac"`

### Building OpenRV after the first time <a name="rebuilding"></a>

https://aswf-openrv.readthedocs.io/en/latest/build_system/config_common_build.html#building-open-rv-after-the-first-time

After building the project for first time, you no longer need to run the bootstrapping step above. Rebuilding the project should now go faster, since you don't have to recompile the dependencies. This might be useful if you want to pull in and have access to future bug fixes and features.

To rebuild it, simply run: `rvmk`