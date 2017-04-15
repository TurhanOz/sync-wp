---
ID: 156
post_title: 'Reactive Directory Chooser &#8211; An Open Source journey'
author: turhan
post_date: 2015-03-27 12:12:54
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/reactive-directory-chooser-an-open-source-journey/
published: true
dsq_thread_id:
  - "3632156753"
---
I am developing applications for years now, but only in professional area, where everything you produced is proprietary, and does not belong to you, but to the client who paids or employs you... So I have decided to publish a simple Android library, just to experiment the journey of open source. But you'll see, this journey is not as marvelous as it looks like, but definitely worth it, especially for the learning process. 
# The library Most of the android applications save data to the external storage of the device. The storage location is often empirically defined by the application itself. It would be better to give back the control to the user; the latter should be able to choose the folder in which he wants to save his data. To do so, the user should be able to navigate through the external storage, create custom folders if he needs to and select the appropriate one for his needs. The library I have created, called Reactive Directory Chooser, answers these needs; It is available on 

[GitHub][1] and [Maven Central][2]. There is, of course, no need to reinvent the wheel, so obviously I checked first. [Others libraries exist][3] out there but were not exactly matching my requirements. 
# The requirements The required functionalities are basic : - listing folder's content (content should be folders only) - navigating from one folder to another - creating folder - notifying client of the selected folder. Based on the current Android Dashboard (march 2015), it worths supporting back from android API 10 (gingerbread, almost 6% of the market) to the latest known API 22. The others available libraries on the market are only supporting starting from honeycomb (API 11), which is sadly a waste. 

# Technical side

## - RxJava (Observers, Observable, Operation) The operations of listing a folder and creating a folder are quite simple. But obviously, these operations have to be processed asynchronously. More over, as we wanted to only display folders (and not regular files), there is here a process of filtering. Technically speaking, Reactive Programming is a proper answer to these operations : indeed, we can easily trigger operation in an IO thread for instance and also filter the stream to keep only folder type files. I am a convinced advocate of Reactive Programming (

[check my conference given in 2014 during AndroidDeveloperDays][4]). Many frameworks exist out there, but the only one I have tested so far is [RxJava][5]. How I proceed ? I isolate the Observable, which produces items asynchronously , from the Observer, that consumes the items in the Android main Thread. These object are encapsulated into Operation objects that trigger the computation on demand, with canceling the previous one if still processing. 
## - Clean code As a developer, I try to continuously improve myself as much as possible, and believe me, producing clean code is not easy but so challenging, a life long journey. The first question I ask myself is "what is the responsibility of this object ?", trying to keep in mind the 

[Single Responsibility Principle][6]. This really is a starting point that will keep your code from being insignicantly polluted. When I review code, from other open source projects for instance, I am really upset/sad/annoyed to see classes that are very long, mixing all kind of non sense operations... This should end! As open source developer, we have a huge responsibility. Our duty is to try to donate to the community good examples of codes, otherwise how will we improve ? Remember, improve every single day if you could. I am not saying the code I produce is clean nor perfect, I'm just saying that I try to make the effort in that way. Another point that I try to give importance is to keep modules of your software independent from each other as much as possible; in other word, [uncoupling objects][7]. For this part, using a Bus, such as [EventBus][8], is a proper answer from my point of view. It lets independent objects producing event and others consuming events. 
## - Coding I first started to develop the backend side, then the UI and finally wiring up all. I like this approach in the fact that they are independent, and force you to properly split your modules. This also gives you the freedom to adapt or change your UI more easily. Also, most of the time, when you deal with UI, you have a great chance to apply the MVC or MVP pattern. Keep in mind that Fragment or Activity are not exactly controller, their main responsability is to deal with lifecycle callbacks as well as saving/restoring ui states (if not delegated to the low level UI widget). 

## - Unit Testing I shouldn't dissociate Unit Test from coding, because Unit Test are important piece of code. On the contrary, I've done that to highlight its importance. Applied in 

[Test Driven Development][9] practice, it gives you a high level of understanding of your object and gives you pointers on how to properly apply Single Responsibility Principle. I am a huge fan of TDD even if, I confess, I'm not applying it everytime, especially when dealing with Android components. However, depending on the quality of the code you want to produce, Unit Test are mandatory. It's also a good documentation for the developer and could prevent from breaking code. So lots of benefits as you can see. For the Unit tests, I am using Robolectric in combination with Mockito. The latest Version of Android Studio as well as the Android Gradle Plugin brings support for Unit Testing within the IDE. However, you need to follow [this process first][10]. As an android developer, I start feeling annoyed when I have constantly to configure my IDE whenever a new release is on the air. Then, you need to explicitly add the roboletric gradle plugin to your root build.gradle file, such as : <pre class="lang:default decode:true ">dependencies {
classpath 'com.android.tools.build:gradle:1.1.0'
classpath 'org.robolectric:robolectric-gradle-plugin:1.0.1'
}</pre> It's not finished; you have to apply the 'org.robolectric' plugin to the build.gradle file of your module, as described in the 

[documentation][11]. Also, in order to let Robolectric find the Manifest and properly plays the tests, we need to annotate our testing classes such as : <pre class="lang:java decode:true ">@RunWith(RobolectricTestRunner.class)
@Config(manifest = "library/src/main/AndroidManifest.xml", emulateSdk = 18)</pre> Finally, if you are a mac user, you need to make another configuration to the IDE, as 

[follow][12]. Some configuration again that is a waste, blocking you to directly create value. 
## - Fresh UI This open source library has been the opportunity to properly implement fresh and interesting components such as RecyclerView, CardView, FloatingActionButton. 

## - Code Review I was working alone on the project. So it is important that you ask to friends or other developer to try to review your code. This gives you a very important feedback that could considerably improve the quality of your code. 

# Continous Integration It's very important, when you change your code (and this will happen a lot) that you prevent your change from integration issues. This could easily be detected, with modern and common practice called 

[Continuous Integration][13]. Of course, with no or few tests on your code, continuous integration is useless. But once set up, it's very convenient. My Open Source library is hosted on github. I decided to go with [TravisCI][14] service, cause it is free, integrated with GitHub and support Android. Travis is very easy to use. After being signed and linked your github account to the service, you just have to add a ".travis.yml" file to your repository such as described in the [documentation][15]. As an example, my travis.yml file looks like : <pre class="lang:yaml decode:true ">language: android
android:
components:
- build-tools-22.0.1
- android-22
- extra-android-support
- extra-android-m2repository</pre> It simply says to use or to install required version of android, extras & so... Very easy. Then, whenever you push a commit to your repository, a travis job will be triggered. 

# Publishing on Maven Central

## - Process Pushing your code publicly to github is one step. To make it available on Maven Central as Open Source Software Repository Hosting (OSSRH) is another step. The process requires few steps that could unfortunately take some time. The precise process is as follow : - First, we need to 

[create an account and a Ticket][16] to provide [project information][17]. This will create you a dedicated groupId. - Then we need to generate signing keys as explained in the [requirement pages][18] . To do so, we need to download and install [gpg][19] - We need then to configure our build.gradle to define the uploadTask, as described [here.][20] - Then, we can upload our AAR module by calling the uploadArchives task : <pre class="lang:sh decode:true ">$ ./gradlew uploadArchives</pre>

<span style="line-height: 1.5;">Finally, once deployed to OSSRH, we need to get the <a href="http://central.sonatype.org/pages/releasing-the-deployment.html">component public</a> by closing and releasing the artifact.</span> As you can see, the process is quite long for the first time. But once set up, you only need to call the uploadArchives task (of course, increase your build version first inside your build.gradle) then make your component public and that's it. 
## - Issues and Solutions

### #1 SemVer When trying to release for the first time, the deployment fails. It was due to the fact that the dependencies of my module was using the 

[Semantic Versioning][21] + notation for the patch, such as : <pre class="lang:default decode:true ">compile 'com.android.support:appcompat-v7:22.0.+'</pre> To fix this failure, we have to explicitly set the version, without permissivity, such as : 

<pre class="lang:default decode:true ">compile 'com.android.support:appcompat-v7:22.0.0'</pre> Unfortunately, by doing this, we loose all the benefits of being permissive with dependency. 

### #2 transitive dependency Once deployed publicly, I try to include the freshly released AAR artifact inside another empty android project, by adding this dependency : 

<pre class="lang:default decode:true ">compile 'com.turhanoz.android:reactivedirectorychooser:0.0.10@aar'</pre> However, no matter what, the dependencies of my dependencies were not present. There is probably an issue between the android studio version vs the gradle version of the wrapper vs the android gradle plugin version... The only solution I have found so far is to explicitely enable transitivity when setting dependency, such as : 

<pre class="lang:default decode:true ">compile ('com.turhanoz.android:reactivedirectorychooser:0.0.10@aar'){       
 transitive=true    
}</pre>

**#3 Unit Testing not working first time** It could happen that the tests are not working on the first launch. This is a [know bug][22]. Simply run on your terminal $ ./gradlew clean test 
# The License Just for fun, I have decided to create my own license instead of going with traditional license such as gpl or so. This new license is called the 

["THE VIRGIN-MOJITO-WARE LICENSE"][23] and is highly inspired from the BeerWare License. 
# Conclusion Woaa! It's done now, I can feel released to have gone through all this process. There is a huge difference between coding on your own computer and hiding the software somewhere in the hard drive, versus making the code available to anyone. Your code belongs now to anyone. The way you code and you designed your application is open to any review. Constructive or not. If you have a big ego, this could be disturbing. But from my point of view, we have to be very humble and I think that any kind of review could only gives you one step forward for self improvement. It has actually been a very pleasant exercice, even if I was frustrated of the development environment. I spend more time setting up environment (especially for testing, deployment and transitive dependency issue) than creating my code, the value. To be honest with you, I probably have spent 3,5 mornings for coding the entire library and sample (from unit test, designing software architecture, coding in a reactive programming paradigm, learning components that I haven't used before (such as RecyclerView) to the entire UI). I hope you'll find useful information in this post and also hope that the issues I've gone through could save you some time. Do not hesitate to use the library, or even better, to review and improve it :)

 [1]: https://github.com/TurhanOz/ReactiveDirectoryChooser
 [2]: http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.turhanoz.android%22%20AND%20a%3A%22reactivedirectorychooser%22
 [3]: https://android-arsenal.com/tag/35
 [4]: http://turhanoz.com/introduction-to-reactive-programming-on-android/
 [5]: https://github.com/ReactiveX/RxJava
 [6]: http://en.wikipedia.org/wiki/Single_responsibility_principle
 [7]: http://en.wikipedia.org/wiki/Coupling_(computer_programming)
 [8]: https://github.com/greenrobot/EventBus
 [9]: http://fr.wikipedia.org/wiki/Test_driven_development
 [10]: http://tools.android.com/tech-docs/unit-testing-support
 [11]: https://github.com/robolectric/robolectric-gradle-plugin
 [12]: https://github.com/robolectric/robolectric/wiki/Running-tests-in-Android-Studio
 [13]: http://en.wikipedia.org/wiki/Continuous_integration
 [14]: https://travis-ci.org
 [15]: http://docs.travis-ci.com/user/languages/android/
 [16]: http://central.sonatype.org/pages/ossrh-guide.html
 [17]: http://central.sonatype.org/pages/producers.html
 [18]: http://central.sonatype.org/pages/requirements.html
 [19]: https://gpgtools.org
 [20]: http://central.sonatype.org/pages/gradle.html
 [21]: http://semver.org
 [22]: https://github.com/robolectric/robolectric/issues/1620
 [23]: https://github.com/TurhanOz/ReactiveDirectoryChooser/blob/master/LICENSE