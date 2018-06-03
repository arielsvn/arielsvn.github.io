---
title: Installing React Native on Ubuntu 16
date: 2016-08-20 09:00:01 Z
categories:
- setup
layout: post
---

These are the steps I followed to configure React Native on Ubuntu 16, a full guide is here https://facebook.github.io/react-native/docs/getting-started.html

## Install NodeJS (used 4.x LTS at the moment)

```console
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## Install Java JDK 

```console
sudo apt-get install openjdk-8-jdk
```

> This throws a warning when you open Android Studio suggesting to install oracle JDK, but should be enough as I'm not planning to use it.

ref: http://askubuntu.com/a/464894/62726


## Install Android Studio

Follow instructions at https://developer.android.com/studio/install.html

### Ensure the right SDK is installed

Go to *Tools > Android > SDK Manager* and check there.

Also Launch the standalone SDK Manager from there, and check if the corresponding version of the build tools is also installed.

If these aren't present, react will trow errors asking for them. Like the one below.


```console
* What went wrong:
A problem occurred configuring project ':app'.
> failed to find Build Tools revision 23.0.1
```


### Add Android SDK, and Android Studio to your path

```console
sudo vi ~/.profile
```

Add the following at the end of that file, and then restart your computer.

```console
# add android SDK paths
export ANDROID_HOME="$HOME/Android/Sdk"
PATH="$HOME/android-studio/bin:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH"
```

in my case I installed android studio and sdk in my home directory.

ref: http://askubuntu.com/a/60221/62726
PATH variables: http://stackoverflow.com/a/4433551/763705



## Install Watchman

Install some required packages first.

```console
sudo apt-get install automake
sudo apt-get install autoconf
```

ref: http://stackoverflow.com/a/33592274/763705


```console
git clone https://github.com/facebook/watchman.git
cd watchman
git checkout v4.6.0  # the latest stable release
./autogen.sh
./configure
make
sudo make install
```

guide https://facebook.github.io/watchman/docs/install.html#installing-from-source



## (Optional) Install Gradle Daemon

If you plan to make changes to Java code https://docs.gradle.org/2.9/userguide/gradle_daemon.html

(I didn't used it)


## Install React native via npm

```console
sudo npm install -g react-native-cli
```


## Install Virtual Box (required by Genymotion)

download from https://www.virtualbox.org/wiki/Linux_Downloads


## Install Genymotion

download from https://www.genymotion.com/download/

And add a new device.




## (Optional) Test installation

```console
react-native init AwesomeProject
cd AwesomeProject
react-native run-android
```





## Solving errors later on

```console
* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: Timeout getting device list.
```

Set the correct SDK url in genymotion *Settings > ADB*

http://stackoverflow.com/a/37932527/763705






# To run the project

On the project root execute `react-native start`, to start the packager and leave it running.

And also `react-native run-android` to run the app.




## Installing an IDE: Nuclide

Install Atom: https://atom.io/download/deb#atom-on-linux

If after initializing atom gives an error:

```
 TypeError: Unable to watch path
```

then increase the number of files watched by inotify, and restart Atom. I used

```
echo 65536 | sudo tee -a /proc/sys/fs/inotify/max_user_watches
```

Install nuclide (it's an Atom package)

```
apm install nuclide
```

After opening Atom, also install the recomended packages. Go to *Packages > Settings View > Manage Packages*


### Install Flow

Instructions here: https://flowtype.org/docs/getting-started.html#installing-flow

```
touch .flowconfig
sudo npm install -g flow-bin
```

Flow will be installed at `/usr/lib/node_modules/flow-bin`



# Running the project from Nuclide

Start the packager *Nuclide > React Native > Start Package*, and then from the console run:

```
react-native run-android
```

