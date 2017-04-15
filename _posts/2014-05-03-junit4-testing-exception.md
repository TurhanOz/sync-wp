---
ID: 13
post_title: 'Junit4 : testing Exception'
author: turhan
post_date: 2014-05-03 21:00:11
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/junit4-testing-exception/
published: true
dsq_thread_id:
  - "2660728175"
---
Junit 4.7 introduces the <a href="http://junit.org/javadoc/latest/org/junit/rules/ExpectedException.html">ExpectedException</a> rule that eases Exception testing.

<strong># the 'yeah, kind of...' way
</strong>I love Square's projects. They are pushing great libs, especially on Android. They push a high level of quality on their code (unit testing, decoupling, readability &amp; so...).

However, I was kind of surprised when I looked into the Phrase project, a single Class project that processes Android String formatting. Awesome project by the way.
The code is quite clean and the test cases seem well relevant, but testing Exception is like :

[gist id=81f79b7a00a2d8c47c4e file=test-with-try-catch]

Ok. it does the job, but the readability could clearly be improved as there are some uncessary extra code here. Adding try/catch blocks into Unit tests is simply not the way to do AMHA. These tests could be enhanced :)

<strong># the partial way</strong>
You can use the <a href="http://junit.org/javadoc/latest/org/junit/Test.html#expected()">expected parameter</a> within the @Test annotation to specify the expected Throwable. The code is clearly more readable than previously:
[gist id=81f79b7a00a2d8c47c4e file=test-with-exptected-annotation]

However, this way has its limit, this is why it is 'partial'. Consider this test :

[gist id=81f79b7a00a2d8c47c4e file=test-expected-annotation-limit]

In this precise test case, you don't know if the Constructor throws the Exception or the following method. You could (and should) of course split the test but let's see some better way.

<strong># a better way</strong>
As briefly evoked in the introduction, Junit 4.7 introduces the ExpectedException rule. The power of this rule lies 1. in covering the previously explained limitation, 2. in gaining in readability and 3. in the ability to not only handle the Exception type, but also the Exception message.
Consider the Square/Phrase test updated with ExpectedException :

[gist id=81f79b7a00a2d8c47c4e file=test-with-ExpectedException]

As you can see, the test is clearly more consize and more readable. You only need to declare the <em>ExpectedException</em> rule in your test class as public. That's it :)

I have submited a <a href="https://github.com/square/phrase/pull/8">pull request</a> to Square/Phrase project by correcting all the Unit Tests dealing with Exceptions so the quality and readability of the code will hopefully benefit to the community.

<strong>To sum up 'Testing Exception' :</strong>
- avoid try/catch blocks within your unit tests
- push readability as much as possible
- use ExpectedException rule to test the Throwable and the associated message