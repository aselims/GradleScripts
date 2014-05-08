GradleScripts
=============

Some useful scripts for our based Gradle compiled Android Apps

<h3>Create relase signed apk file</h3>

```
...
signingConfigs {
    release {
        storeFile file(System.console().readLine("\n\$ Enter keystore path: "))
        storePassword System.console().readPassword("\n\$ Enter keystore password: ")
        keyAlias System.console().readLine("\n\$ Enter key alias: ")
        keyPassword System.console().readPassword("\n\$ Enter key password: ")
    }
}
...
```
This will prompt for each of the parameters.

Having said this, in these situations, you are better off setting environment variables for these parameters and using them in the gradle file. Environment variables can be accessed with System.getenv("<VAR-NAME>")
```
... 
signingConfigs {
    release {
        storeFile file(System.getenv("KEYSTORE"))
        storePassword System.getenv("KEYSTORE_PASSWORD")
        keyAlias System.getenv("KEY_ALIAS")
        keyPassword System.getenv("KEY_PASSWORD")
    }
}
...
```


<h3>Controlling Android properties of all your modules from the main project</h3>
If you have a lot of Android modules, you may want to avoid manually setting the same values in all of them. Because you probably have a mix of android and android-library project you can't apply these plugins through a subprojects closure. However you can set the value on the root project and then reference this from the modules. For example:

in the root project's build.gradle:

```
ext {
  compileSdkVersion = 19
  buildToolsVersion = "19.0.1"
}
```
in all the android modules:
```
android {
  compileSdkVersion rootProject.ext.compileSdkVersion
  buildToolsVersion rootProject.ext.buildToolsVersion
}
```

This way you only ever need to update one build.gradle. 

Note that ext.* properties are dynamically created so you can add new properties of any type depending on your needs. You can then extend this to any other properties shared by some or all of your projects (minSdkVersion, targetSdkVersion, etc...)

<h3>Improving Build Server performance</h3>
The Gradle based build system has a strong focus on incremental builds. One way it is doing this in doing pre-dexing on the dependencies of each modules, so that each gets turned into its own dex file (ie converting its Java bytecode into Android bytecode). This allows the dex task to do less work and to only re-dex what changed and merge all the dex files.

While this is great for incremental builds, especially when running from the IDE, this makes the first compilation slower. In general build system will always perform clean builds and this pre-dexing becomes a penality. Since there will not be any incremental builds, it is really not needed to use pre-dexing.

Here's how to disable it only on your build server. First, in your root build.gradle add the following:

```
project.ext.preDexLibs = !project.hasProperty('disablePreDex')

subprojects {
    project.plugins.whenPluginAdded { plugin ->
        if ("com.android.build.gradle.AppPlugin".equals(plugin.class.name)) {
            project.android.dexOptions.preDexLibraries = rootProject.ext.preDexLibs
        } else if ("com.android.build.gradle.LibraryPlugin".equals(plugin.class.name)) {
            project.android.dexOptions.preDexLibraries = rootProject.ext.preDexLibs
        }
    }
}
```
Then configure your build server to call Gradle with:
```
./gradlew clean assemble -PdisablePreDex
```



