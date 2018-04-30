# Xamarin.iOS libary build on Linux

There is no official Xamarin.iOS build for Linux. However, at least compiling Xamarin.iOS libaries works with this:

- From a Windows machine with VSStudio 2017 and Xamarin.iOS, copy:

"C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\MSBuild\Xamarin\iOS"
to
"/usr/lib/mono/xbuild/Xamarin/iOS"
(plus the props and target files)

- Open /iOS/Xamarin.iOS.Common.targets and add at line 87:

`<PropertyGroup>`  
    `<TargetFrameworkRootPath>/usr/lib/mono/xbuild-frameworks/</TargetFrameworkRootPath>`   
`</PropertyGroup>`

- Open /iOS/Xamarin.iOS.CSharp.After.targets: Comment out / delete property group or try to delete whole file. (For some reason the content of this file changes the CSC path which is wrong when using Mono under Linux.)

- From a Windows machine with VSStudio 2017 and Xamarin.iOS, copy:

"C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\Common7\IDE\ReferenceAssemblies\Microsoft\Framework\Xamarin.iOS"
to
"/usr/lib/mono/xbuild-frameworks/Xamarin.iOS"
