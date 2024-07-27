# WLA
Windows-Linux-Android SFML project

Project template for Windows, Linux, and Android.

Compilation for Windows and Linux can be done with Visual Studio.

Linux compilation is done with WSL in Visual Studio.

Compilation for Android can be done with Android Studio.

## Setup

All external assets (images, audio, fonts) must be placed inside the `assets/` folder.

The `src/` folder is used for `.cpp`, `.h`, and optionally `.rc` files. 

You'll need to specify your SFML directories for each architecture in CMakeLists.txt

The `versions/` folder is ignored by Git and can be used for saving previous releases.

The `res/` folder is used by Android Studio for icons and styles.
