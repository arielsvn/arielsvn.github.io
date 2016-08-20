---
layout: post
title:  "Installing React Native on Ubuntu 16"
date:   2016-08-20 09:00:01
categories: setup
---

Steps to configure React Native on Ubuntu 16

These are the steps I followed, a full guide is here https://facebook.github.io/react-native/docs/getting-started.html

## Install NodeJS (used 4.x LTS at the moment)

```
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## Install Java JDK 

```
sudo apt-get install openjdk-8-jdk
```

> This throws a warning when you open Android Studio sugesting to install oracle JDK, but should be enough as I'm not planning to use it.

ref: http://askubuntu.com/a/464894/62726


## Install Android Studio

Follow instructions at https://developer.android.com/studio/install.html

### Ensure the right SDK is installed

Go to *Tools > Android > SDK Manager* and check there.

Also Launch the standalone SDK Manager from there, and check if the corresponding version of the build tools is also installed.

If these aren't present, react will trow errors asking for them. Like the one below.


```
* What went wrong:
A problem occurred configuring project ':app'.
> failed to find Build Tools revision 23.0.1
```


### Add Android SDK, and Android Studio to your path

```
sudo vi ~/.profile
```

Add the following at the end of that file, and then restart your computer.

```
# add android SDK paths
export ANDROID_HOME="$HOME/Android/Sdk"
PATH="$HOME/android-studio/bin:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH"
```

in my case I installed android studio and sdk in my home directory.

ref: http://askubuntu.com/a/60221/62726
PATH variables: http://stackoverflow.com/a/4433551/763705



## Install Watchman

Install some required packages first.

```
sudo apt-get install automake
sudo apt-get install autoconf
```

ref: http://stackoverflow.com/a/33592274/763705


```
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

```
sudo npm install -g react-native-cli
```


## Install Virtual Box (required by Genymotion)

download from https://www.virtualbox.org/wiki/Linux_Downloads


## Install Genymotion

download from https://www.genymotion.com/download/

And add a new device.




## (Optional) Test installation

```
react-native init AwesomeProject
cd AwesomeProject
react-native run-android
```





## Solving errors later on

```
* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: Timeout getting device list.
```

Set the correct SDK url in genymotion *Settings > ADB*

http://stackoverflow.com/a/37932527/763705






# To run the project

On the project root execute `react-native start`, to start the packager and leave it running.

And also `react-native run-android` to run the app.




