# OkBuck
[ ![Download](https://api.bintray.com/packages/okbuild/maven/OkBuild/images/download.svg) ](https://bintray.com/okbuild/maven/OkBuild/_latestVersion)
[![Master branch build status](https://travis-ci.org/OkBuilds/OkBuck.svg?branch=master)](https://travis-ci.org/OkBuilds/OkBuck)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-OkBuck-green.svg?style=flat)](https://android-arsenal.com/details/1/2593)

OkBuck is a gradle plugin, aiming to help developers utilize the fast build 
system: [Buck](https://buckbuild.com/), based on an existing project built using gradle. This plugin lets you have both build systems work side by side.

[Wiki](https://github.com/OkBuilds/OkBuck/wiki), 
[中文版](https://github.com/OkBuilds/OkBuck/blob/master/README-zh.md)

## Why?
Gradle is typically the default build tool for android development, and 
to migrate to buck, there is a lot of overhead, which can be difficult and 
buggy. OkBuck automates this migration with very few lines of configuration.

## Installation
In root project `build.gradle` file:

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.github.okbuilds:okbuild-gradle-plugin:0.2.2'
    }
}

apply plugin: 'com.github.okbuilds.okbuck-gradle-plugin'
```

That's all for basic case, no need for any other configuration. After appling the plugin, these tasks are added
  +  `okbuck` will generate BUCK files
  +  `okbuckClean` will **delete all** files/dirs generated by OkBuck and BUCK
  +  `installBuck` will download and install buck and symlink it to `./buck` in your project

You can type `buck targets` to get a list of targets that can be build. The generated `.buckconfig.local` file will have some aiases setup to build your apps without having to type the rulename. i.e you can do things like `buck build appDebug another-appPaidRelease` etc. The plugin also generates an empty `.buckconfig` file if it does not exist. You can customize the settings in the `.buckconfig` file by using the various [options](https://buckbuild.com/concept/buckconfig.html)

## Configuration
```gradle
okbuck {
    buildToolVersion "23.0.3"
    target "android-23"
    linearAllocHardLimit = [
            app: 7194304
    ]
    primaryDexPatterns = [
            app: [
                    '^com/github/okbuilds/okbuck/example/AppShell^',
                    '^com/github/okbuilds/okbuck/example/BuildConfig^',
                    '^android/support/multidex/',
                    '^com/facebook/buck/android/support/exopackage/',
                    '^com/github/promeg/xlog_android/lib/XLogConfig^',
                    '^com/squareup/leakcanary/LeakCanary^',
            ]
    ]
    projectTargets = [
            'app': 'devDebug',
            'common': 'freeRelease',
            'dummylibrary': 'freeRelease',
    ]
    exopackage = [
            appDebug: true
    ]
    annotationProcessors = [
            "local-apt-dependency": ['com.okuilds.apt.ExampleProcessor']
    ]
    appLibDependencies = [
            'appProd': [
                    'buck-android-support',
                    'com.android.support:multidex',
                    'libraries/javalibrary:main',
                    'libraries/common:paidRelease',
            ],
            'appDev': [
                    'buck-android-support',
                    'com.android.support:multidex',
                    'libraries/javalibrary:main',
                    'libraries/common:freeDebug',
            ],
            'appDemo': [
                    'buck-android-support',
                    'com.android.support:multidex',
                    'libraries/javalibrary:main',
                    'libraries/common:paidRelease',
            ]
    ]
    buckProjects = project.subprojects
    keep = []

    install {
        gitUrl 'https://github.com/OkBuilds/buck.git'
        sha 'okbuck'
    }
}
```

+  `buildToolVersion` specifies the version of the Android SDK Build-tools, defaults to `23.0.3`
+  `target` specifies the Android compile sdk version, default is `android-23`
+  `linearAllocHardLimit` and `primaryDexPatterns` are maps, configuration used by buck for multidex. For more details about multidex configuration, please read the
[Multidex wiki page](https://github.com/OkBuilds/OkBuck/wiki/Multidex-Configuration-Guide), 
if you don't need multidex, you can ignore these parameters
+  `projectTargets` is a map of project name to the buck target to use when generating an IntelliJ project.
+  `exopackage`, `appClassSource` and `appLibDependencies` are used for
configuring buck's exopackage mode. For more details about exopackage configuration, 
please read the [Exopackage wiki page](https://github.com/OkBuilds/OkBuck/wiki/Exopackage-Configuration-Guide), if you don't need exopackage, you can ignore these parameters
+ `annotationProcessors` is used to depend on annotation processors declared locally as another gradle module in the same root project.
+  `buckProjects` is a set of projects to generate buck configs for. Default is all sub projects of the root project.
+ `keep` is a list of files to not clean up by the plugin when running `okbuckclean`. This may be useful to keep the `buck-out` folder around for faster incremental builds even when buck files are regenerated. Also useful if you want made manual modifications to some buck configuration and would like to keep it intact while regenerating the configuration for other projects.
+ `install` is used to download and install buck from the [official version](https://github.com/facebook/buck) or from any fork. It keeps a cache of different forks you can use across your projects. To install the specified version you need to explictly run the `buckInstall` task and can use it via the `buck` symlink in your project.
 - `gitUrl` - The git url of the buck fork. Default is [facebook/buck](https://github.com/facebook/buck)
 - `sha` - The git sha/branch/tag to checkout before building. Defaults to origin/master
 - `dir` - The directory to clone buck in. Defaults to `~/.gradle/caches/okbuilds`
+ The keys used to configure various options can be either for 
 - All buildTypes and flavors i.e `app`
 - All buildTypes of a particular flavor i.e 'appDemo'
 - All flavors of a particular buildType i.e 'appDebug'
 - A particular variant (buildType + flavor combination) i,e 'appDemoRelease'

## Common Issues
+ If you use ndk filters in your build.grade, you must set the `ANDROID_NDK` environment variable pointing to your local android ndk root dir, otherwise BUCK build will fail.
+ If your project uses gradle 2.4, youneed force jdk version to 1.7, [ref1](http://stackoverflow.com/a/21212790/3077508) 
and [ref2](http://stackoverflow.com/a/18144853/3077508)
+ OkBuck aims to provide almost all the features that the android gradle plugin provides,but your project may still be incompatible with buck for various reasons listed on the [Known caveats wiki page](https://github.com/OkBuilds/OkBuck/wiki/Known-caveats). 

## Compatibility
OkBuck is tested under `gradle` 2.2.1 ~ 2.13, and `com.android.tools.build:gradle` 1.5.0 ~ 2.1.0.
If other versions have any compatibility problems, please file an issue.

## [Full Change log](https://github.com/OkBuilds/OkBuck/blob/master/CHANGELOG.md)

## Liscense
```
The MIT License (MIT)

Copyright (c) 2015 OkBuilds

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
