---
ID: 332
post_title: >
  Android SDK Tools vs Platform Tools vs
  Build Tools
author: turhan
post_date: 2015-09-19 17:27:32
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/android-sdk-tools-vs-platform-tools-vs-build-tools/
published: true
---
<p class="p1"><span class="s1">Well, in order to avoid being confused if you are and android developer, this is just a reminder that hopefully will makes things more clear. </span></p>
<p class="p1"><span class="s1">Typically, SDK Tools are platform independent whereas Platform tools, obviously, support the features of the latest Android Platform. Build Tools gather tools that will produce your signed Apk.</span></p>
<p class="p3"><span class="s2">you can get more detailed information from the <a href="https://developer.android.com/tools/help/index.html">official documentation</a>. </span></p>

<h2 class="p1"><span class="s1">SDK Tools</span></h2>
<p class="p1"><span class="s1">Roughly, SDK tools include all the tools that you need to develop for android; </span></p>
<p class="p1"><span class="s1">That includes the SDK manager (to download android SDKs), development tools such as AVD (android virtual devices), Image tools (such as draw 9 patch), Debugging tools (such as ddms), monkey runner and so…</span></p>
<p class="p1"><span class="s1">It is not an exhaustive list of course. You can open the folder on your machine to get all of the tools.</span></p>

<h2 class="p1"><span class="s1">Platform tools</span></h2>
<p class="p1"><span class="s1">These tools are updated for each new android sdks release and is backward compatible. The platform tools include for instance ADB which is a bridge that allows you to communicate with your android device or emulator. ADB has to know how to communication with the Android version that is hosted on your device.</span></p>

<h2 class="p1"><span class="s1">Build Tools</span></h2>
<p class="p1"><span class="s1">The build-tools are used to build your application, that is to produce your APK.
</span><span class="s2">Release notes <a href="https://developer.android.com/tools/revisions/build-tools.html">here</a> and nicely detailed build process <a href="https://developer.android.com/sdk/installing/studio-build.html#detailed-build">here</a>.</span></p>