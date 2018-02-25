# Xamarin.Android On Debian (22.02.2018)

## Mono

- Have latest mono installed from [here](http://www.mono-project.com/download/stable/#download-lin)
- Make sure **msbuild** is working with some sample solution from any path
- Make sure it also works with PCLs (install referenceassemblies-pcl package)

## Android SDK

- Download sdk tools from https://developer.android.com/studio/index.html (command line tools at the bottom)
- Create directory /usr/lib/android-sdk
- Unpack sdk tools into /usr/lib/android-sdk so that /usr/lib/android-sdk/android-sdk-tools/ exists
- run /usr/lib/android-sdk/android-sdk-tools/bin/sdkmanager like e.g.:  
   sdkmanager "build-tools;23.0.3" "platforms;android-23" "platform-tools" for Android 6.0
- verify with sdkmanager --list that packages are installed
- They are now all under /usr/lib/android-sdk (one directory up from the sdk tools) which is the  android sdk directory
 
## Java 8

(not 100% sure about these steps)

- It did not work with Java 9
- apt-get install software-properties-common
- add-apt-repository ppa:webupd8team/java
- apt-key add [some key file I cannot find anymore]
- apt-get update
- apt install oracle-java8-set-default

## Xamarin.Android

- Download Xamarin.Android Linux **Build** [here](https://jenkins.mono-project.com/view/Xamarin.Android/job/xamarin-android-linux/lastSuccessfulBuild/Azure/) and unpack e.g. under /tmp
- Copy Xamarin subdirectory recursively from:  
    /tmp/xamarin.android-oss_vx.x.xx.x_Linux-x86_64_HEAD_xxxxxxxx/bin/Release/lib/xamarin.android/xbuild to /usr/lib/mono/xbuild 
- Copy MonoAndroid subdirectory recursively from:  
    /tmp/xamarin.android-oss_vx.x.xx.x_Linux-x86_64_HEAD_xxxxxxxx/bin/Release/lib/xamarin.android/xbuild-frameworks to /usr/lib/mono/xbuilds-frameworks

## Build

After a lot of reading through /usr/lib/mono/xbuild/Xamarin/Android/Xamarin.Android.Common.targets two important parameters turn 
up:

- *TargetFrameworkRootPath* must be set to /usr/lib/mono/xbuild-frameworks/ (see copy before) There is some bug in the build code which requires the trailing slash
- *AndroidSdkDirectory* must be set to /usr/lib/android-sdk

Specified Target Framework in project file must be installed with android sdk tools

Change directory to where solution is and run:

 msbuild /p:TargetFrameworkRootPath="/usr/lib/mono/xbuild-frameworks/" /p:AndroidSdkDirectory="/usr/lib/android-sdk"

If that works put properties into /usr/lib/mono/xbuild/Xamarin/Android/Xamarin.Android.Common.targets with:  

`<PropertyGroup>`  
    `<TargetFrameworkRootPath>/usr/lib/mono/xbuild-frameworks/</TargetFrameworkRootPath>` 
    `<AndroidSdkDirectory>/usr/lib/android-sdk</AndroidSdkDirectory>`  
`</PropertyGroup>`

and just call msbuild without parameters

