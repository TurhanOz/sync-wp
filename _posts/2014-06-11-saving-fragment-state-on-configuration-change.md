---
ID: 58
post_title: >
  Saving Fragment State on Configuration
  Change
author: turhan
post_date: 2014-06-11 22:00:51
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/saving-fragment-state-on-configuration-change/
published: true
dsq_thread_id:
  - "2757531140"
---
<strong>prerequisite</strong>
- <a href="http://developer.android.com/guide/components/fragments.html">Fragment Component</a>
- <a href="http://developer.android.com/guide/components/fragments.html#Lifecycle">Fragment lifecycle</a>

<strong>What I needed :</strong>
As you know (if not, please read the prerequisite first), the Fragment lifecycle is highly tied to the activity lifecycle. That said, my need was simply to retain some state of my Fragment and restore it later on after a recreation. A typical scenario is when user rotates screen.
To do so, one needs to override <a href="http://developer.android.com/reference/android/app/Fragment.html#onSaveInstanceState(android.os.Bundle)">onSaveInstanceState</a> method of the Fragment and save any value into a Bundle, as follow :
<pre class="lang:java decode:true">@Override
public void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putParcelable("myParcelableObject", myParcelableObject);
}</pre>
The saved Bundle could be retrieved during either onCreate(), onCreateView(), onViewCreated(), onActivityCreated() or onViewStateRestored() callbacks.

<strong>What was the problem ?</strong>
In my case, the saved bundle was always null in any of the callbacks listed above, although I properly called onSaveInstanceState().

<strong>How to solve it ?</strong>
Firstly, my Activity's layout looked like this :
<pre class="lang:xhtml decode:true">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="fill_parent"
android:layout_height="fill_parent"&gt;

&lt;fragment android:name="com.example.MyFragment"
android:layout_width="fill_parent"
android:layout_height="fill_parent" /&gt;

&lt;/RelativeLayout&gt;</pre>
<strong>What was missing here is an android:id field</strong> within my fragment so Android could recognize the stored Fragment! We simply correct the layout as follow :
<pre class="lang:default decode:true">&lt;fragment android:name="com.example.MyFragment"
android:id="@+id/myfragment"
android:layout_width="fill_parent"
android:layout_height="fill_parent" /&gt;</pre>
As described in the <a href="http://developer.android.com/guide/components/fragments.html#Adding">documentation</a>,
<blockquote>Each fragment requires a unique identifier that the system can use to restore the fragment if the activity is restarted (and which you can use to capture the fragment to perform transactions, such as remove it). There are three ways to provide an ID for a fragment:
Supply the android:id attribute with a unique ID.
Supply the android:tag attribute with a unique string.
If you provide neither of the previous two, the system uses the ID of the container view.</blockquote>
<strong>Conclusion</strong>
In a word, always feed your fragment with an identifier !