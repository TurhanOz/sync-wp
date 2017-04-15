---
ID: 54
post_title: 'What are you doing Google ? #3'
author: turhan
post_date: 2014-06-09 22:00:18
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/what-are-you-doing-google-3/
published: true
dsq_thread_id:
  - "2750883748"
---
This is the third post of a looooong list regarding what I do not agree with the Android source code. I have the chance to create Android applications, so I regularly read codes, good ones as well as bad ones... **What is my goal of doing so **? I give myself the freedom to criticize what Google do because, as a developer, we have to improve days after days! Improving means not only practicing, but also exploring and studying quality open source projects. So either we have to study the right stuff, or we have to highlight what is wrong! Btw, don't misunderstand me, I like Google projects ! **Naming consistency** Let's talk in this post about naming consistency. To do so, we will take the [MotionEvent][1] class from the android project. As stated in the documentation, *"[MotionEvent is] used to report movement (mouse, pen, finger, trackball) events"* I am currently writing another post which will deeply describe the android touch mechanism (some teasing now :) but here is the big picture : A MotionEvent holds an action (down, up, move...). It also holds a set of axis values which represent individual fingers position(in multi touch environment). Each finger in the set has an unique id, called pointerId. However, each finger in that set has also an index that could vary within 2 MotionEvents. So as you can second guess, the MotionEvent object exposes methods to get the pointerId (unique value) from a varying pointerIndex. This method is the following : public final [int getPointerId (int pointerIndex)][2] Here, the method is quiet clean and explicite : it requires a pointerIndex as a parameter and returns the pointerId. So your sense of developer will naturaly guide you to look for a method called getPointerIndex() from the MotionEvent object, so you could feed the previous getPointerId(int pointerIndex) method. **Guess what? No such method!** At least, no such method having this name. Yeah, why being simple and consistent when we can complexify things? Just dig a bit and you will find a method called public final int [getActionIndex ()][3] The documentation says : *"[getActionIndex ()] returns the associated pointer index."* Well, this is exactly what we are looking for but why on earth the name of the method (actionIndex) is unconsistent with the value returned (pointerIndex) as well as from the documentation ? This is all the more frustrating as the parameter of the previous method needs this value named differently. So, the code we need is the following : i*nt pointerIndex = event.getActionIndex();* * int pointerId = event.getPointerId(pointerIndex) ;* A good code is a code which has lots of specificities. Readability, naming, consistency. As much as possible, be rigorous with yourself and don't neglect these simple little naming consistencies. **What do you think ?**

 [1]: http://developer.android.com/reference/android/view/MotionEvent.html
 [2]: http://developer.android.com/reference/android/view/MotionEvent.html#getPointerId(int)
 [3]: http://developer.android.com/reference/android/view/MotionEvent.html#getActionIndex()