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
- For some reason e.g. the aapt tool does not work with "no such file or directory" - Run: apt-get install lib32stdc++6 lib32z1

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

and just call msbuild without parameters. AndroidSdkDirectory must not contain any placeholder or environment variables. Also ~ for home will not work.

## Build APK

- Seems like when building for all ABIs (armv7,v8,intel and so on) Xamarin on Windows just empties the corresponding XML element which seems to be interpreted as default armv7 only on linux. Need to explicitly specify all wanted ABIs in csproj for all build types (`<AndroidSupportedAbis`>armeabi;armeabi-v7a;x86;arm64-v8a`</AndroidSupportedAbis`> Leaving x64 out in this example). Otherwise app cannot be installed on devices with some "corrupted" error message. 
- Xamarin offers /t:SignAndroidPackage which unfortunately needs key store location and passwords as part of the csproj file. Security no-go for me. Better use /t:PackageForAndroid which builds APK only.
- /t:PackageForAndroid does not run zipalign and signing.
- Build for Debug and Release and do debug and release signing by yourself.

### Check signature

jarsigner -verify -verbose -certs my_application.apk

## Remarks

- With version 8.3.99 /t:SignAndroidPackage creates a debug keystore with a signing key which is no longer accepted as safe enough. [Details here.](https://github.com/xamarin/xamarin-android/issues/1361)
