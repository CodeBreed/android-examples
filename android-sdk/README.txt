===============================================================================
                                ANDROID EXAMPLES
                         Code examples written in Scala
                        and adapted from the Android SDK
===============================================================================

This document describes code examples written in Scala and adapted from the
Java examples available together with the Android SDK from Google Inc.
(http://developer.android.com/sdk/). Both the Java source code and the Scala
source code are licensed under the Apache License, Version 2.0.

For information about Scala as a language, you can visit the web site
http://www.scala-lang.org/

In the following we describe the build and installation of our Android
applications written in Scala (and in Java) using the Apache Ant build tool.
Note that the build instructions below apply to both the Unix and Windows
environments.

All Android examples have been run successfully on the virtual Android device
"API_10" configured as follows: 2.3.3 target, 256M SD card and HVGA skin
(for more details see the documentation page
$ANDROID_SDK_ROOT/docs/guide/developing/tools/avd.html).

NB. The file INSTALL.txt gives VERY USEFUL informations about the small
development framework (directories "bin/" and "configs/") included in this
software distribution; in particular it permits to greatly reduce the build
time of Android applications written in Scala.


Software Requirements
---------------------

In order to build/run our Android examples we need to install the following
free software distributions (tested versions and download sites are given in
parenthesis) :

1) Sun Java SDK 1.6 or newer (1.7.0_03   , www.sun.com/java/jdk/)
2) Scala SDK 2.7.5 or newer  (2.10.0     , www.scala-lang.org/downloads/)
3) Android SDK 9 or newer    (19         , developer.android.com/sdk/)
4) Apache Ant 1.7.0 or newer (1.8.3      , ant.apache.org/)
5) ProGuard 4.4 or newer     (4.7        , proguard.sourceforge.net/)

NB. In this document we rely on Ant tasks featured by the Scala SDK, the
Android SDK and the ProGuard shrinker and obfuscator tool (we will say more
about ProGuard when we look at the modified Ant build script).


Project Structure
-----------------

The project structure of an Android application follows the directory layout
prescribed by the Android system (for more details see the documentation page
$ANDROID_SDK_ROOT/docs/guide/developing/other-ide.html#CreatingAProject).

In particular:

* The "AndroidManifest.xml" file contains essential information the Android
  system needs to run the application's code (for more details see the docu-
  mentation page $ANDROID_SDK_ROOT/docs/guide/topics/manifest/manifest-intro.html)

* The "ant.properties" file defines customizable Ant properties for the
  Android build system; in our case we need to define at least the following
  properties (please adapt the respective values to your own environment):

  Unix:                                   Windows:
    sdk.dir=/opt/android-sdk-linux_x86      sdk.dir=c:/Progra~1/android-sdk-windows
    scala.dir=/opt/scala                    scala.dir=c:/Progra~1/scala
    proguard.dir=${sdk.dir}/tools/proguard  proguard.dir=${sdk.dir}/tools/proguard

* The "default.properties" file defines the default API level of an Android
  (for more details see the documentation page
  $ANDROID_SDK_ROOT/docs/guide/appendix/api-levels.html).

* The "build.xml" Ant build script defines targets such as "clean", "install"
  and "uninstall" and has been slightly modified to handle also Scala source
  files. Concretely, we override the default behavior of the "-post-compile"
  target and modify its dependency list by adding the imported target
  "-post-compile-scala":

    <import file="build-scala.xml"/>

    <!-- Converts this project's .class files into .dex files -->
    <target name="-post-compile" depends="-post-compile-scala" />

* The "build-scala.xml" Ant build script defines the targets "compile-scala"
  and "-post-compile-scala" where respectively the "<scalac>" Ant task generates
  Java bytecode from the Scala source files and the "<proguard>" task creates a
  shrinked version of the Scala standard library by removing the unreferenced
  code (see next section for more details). Those two tasks are featured by
  the Scala and ProGuard software distributions respectively.


Project Build
-------------

We assume here the Android emulator is up and running; if not we start it
using the shell command (let us assume the existence of the "API_10"
virtual device) :

   android-sdk> emulator -no-boot-anim -no-jni -avd API_10 &

Then we move for instance to the "Snake" project directory and execute one of
the following Ant targets :

   android-sdk> cd Snake
   Snake> ant clean
   Snake> ant compile-scala
   Snake> ant debug
   Snake> ant installd
   (now let us play with our application on the emulator !)
   Snake> ant uninstall


===============================================================================

Signing your Applications
-------------------------
All Android applications must be signed. The system will not install an
application that is not signed. You can use self-signed certificates to sign
your applications. No certificate authority is needed. For example:

$ keytool -genkey -v -keystore ~/.android/release.keystore
          -alias android-sdk -keyalg RSA -validity 10000


Note about ProGuard
-------------------

The main issue when building an Android application written in Scala is
related to the code integration of the Scala standard library into the
generated Android bytecode. Concretely, we have two choices :

1) We bundle (or better to say -- see the note below --, we try to bundle)
   the full Scala library code (an external library) together with our Android
   application as prescribed by the Android system (only system libraries can
   exist outside an application package).
   In Scala 2.8 the "scala-library.jar" library file has a size of about 5659K;
   it means that even the simplest Android application written in Scala would
   have a respectable foot print of at least 6M in size !

   NB. At this date (May 2010) we could not generate Android bytecode for the
   Scala standard library using the dx tool of the Android SDK. Thus, the
   execution of the following shell command fails with the error message
   "trouble writing output: format == null" :

   /tmp> dx -JXmx1024M -JXms1024M -JXss4M --no-optimize --debug --dex
         --output=/tmp/scala-library.jar /opt/scala/lib/scala-library.jar

2) We find a (possibly efficient) way to shrink the size of the Scala standard
   library by removing the library code not referenced by our Android
   application. Our solution relies on ProGuard, a free Ant-aware obfuscator
   tool written by Eric Lafortune; the ProGuard shrinker is fast and generates
   much smaller Java bytecode archives.

   Application     <myapp>.jar   Scala library classes    classes.dex
   (in Scala)                    (orig. 4151 classes)  (Android bytecode)
   -----------------------------------------------------------------------
   ApiDemos          1169K           409 classes             872K
   ContactManager     362K           362 classes             286K
   CubeLiveWallpaper  351K           353 classes             279K 
   FileBrowser        438K           433 classes             346K
   GestureBuilder      44K           403 classes             337K
   HelloActivity        3K             0 classes               2K
   Home               385K           365 classes             314K
   JetBoy             380K           369 classes             311K
   LunarLander        431K           427 classes             351K
   NotePad            356K           356 classes             282K
   PhoneDialer        326K           349 classes             256K
   SearchableDict     380K           380 classes             301K
   Snake              447K           428 classes             353K

   We now compare the build times and sizes of our Android applications
   (written in Scala) with the orginal examples (written in Java) from the
   Android distribution :

   Application        classes.dex   <app>-debug.apk(1)  build time(2)
                    (Scala / Java)   (Scala / Java)    (Scala / Java)
   ------------------------------------------------------------------
   ApiDemos           872K / 468K     2688K / 2174K     1m18s / 18s
   ContactManager     286K /  17K      127K /   25K       37s /  5s
   CubeLiveWallpaper  297K /  15K      120K /   19K       38s /  5s
   GestureBuilder     337K /  20K      144K /   28K       40s /  5s
   Home               314K /  32K      359K /  247K       40s /  6s
   JetBoy             311K /  24K     1647K / 1530K       47s / 13s
   LunarLander        351K /  18K      250K /  120K       39s /  5s
   NotePad            282K /  21K      132K /   48K       38s /  5s
   SearchableDict     301K /  15K      153K /   44K       45s /  4s
   Snake              353K /  14K      147K /   18K       46s /  4s

   (1) Sizes of application packages include bytecode and resources.
   (2) Elapsed times for Scala builds include ProGuard processing time.

   NB. The above results were measured with ProGuard 4.5.1 on a 2.0 GHz
   Pentium M with 2 GB of memory, using Sun JDK 1.6.0_26 and Scala 2.9.0.1
   on Ubuntu 8.04 Linux.


Have fun!
Stephane

