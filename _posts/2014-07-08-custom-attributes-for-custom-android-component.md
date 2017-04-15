---
ID: 66
post_title: >
  Custom attributes for custom Android
  Component
author: turhan
post_date: 2014-07-08 20:30:26
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/custom-attributes-for-custom-android-component/
published: true
dsq_thread_id:
  - "2829545357"
---
Let's suppose you need to create a custom view or a custom fragment. These components could be inflated by xml or instancited programmatically. In case they are inflated, you could have the need to pass custom attributes to define a initial state of your component. For example : you wanted to create a custom [Fragment][1] (call it MyFragment) that manipulates a *string* and an *integer*. **#1. declare your attributes in res/attrs.xml as follow:** <pre class="lang:xml decode:true ">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;resources&gt;
&lt;declare-styleable name="MyFragment"&gt;
    &lt;attr name="company" format="string"/&gt;
    &lt;attr name="age" format="integer"/&gt;
&lt;/declare-styleable&gt;
&lt;/resources&gt;</pre> As you can see, each attribute requires a format, which can be one of the followings : 

<pre class="lang:default decode:true">boolean
color
dimension
enum
flag
float
fraction
integer
reference
string</pre>

** #2. In you layout (where you declare your fragment), add your custom attributes to your component:** <pre class="lang:default decode:true">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:options="http://schemas.android.com/apk/res-auto"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"&gt;

    &lt;fragment android:name="com.test.fragment.MyFragment"
        android:id="@+id/map"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        options:company_age="20"
        options:company_name="corp"/&gt;
&lt;/RelativeLayout&gt;</pre> There, you need to be carefull to add a new namespace : 

<pre class="lang:default decode:true ">xmlns:options="http://schemas.android.com/apk/res-auto"</pre> The value of the namespace (

*'options'*) could be anything you want. **#3. Extract these attributes from your fragment :** If you check [the Fragment API ][1]reference, the only callback where you could deal with custom attributes is the [onInflate()][2] method As described, this callback should only be responsible for extracting the attributes : 
> " all you should do here is parse the attributes and save them away." Let's suppose I have a Business Object called Company <pre class="lang:java decode:true">public class Company{
    final String name;
    final int age;
}</pre> All I have to do is parsing the attributes and creating my Company BO within onInflate() method as follow: 

<pre class="lang:java decode:true  ">@Override
public void onInflate(Activity activity, AttributeSet attrs, Bundle savedInstanceState) {
    super.onInflate(activity, attrs, savedInstanceState);
    TypedArray typedArray = activity.obtainStyledAttributes(attrs, R.styleable.MyFragment);
    String companyName = typedArray.getString(R.styleable.MyFragment_company_name);
    int companyAge= typedArray.getInt(MyFragment_company_age, 0);

    company = new Company(companyName, companyAge);
    typedArray.recycle();
}</pre>   

**#4. Testing** I should have started by writing in [TDD ][3]but I wanted to describe the attributes definition and parsing first. The following Unit Tests will use [Junit4][4], [Mockito ][5]as well as [Robolectric][6] (tested with 2.2). The [onInflate()][2] method needs an activity instance to properly fetch the attributes. So we will create an Activity instance first. <pre class="lang:java decode:true  ">@Before
public void setUp() throws Exception {
    activity = createActivityControllerAndHostActivity();
    myFragment = new MyFragment();
    spyMyFragment = spy(myFragment);
}

private FragmentActivity createActivityControllerAndHostActivity() {
    controller = Robolectric.buildActivity(FragmentActivity.class);
    return (FragmentActivity) controller.create().start().resume().get();
}

@After
public void tearDown() throws Exception {
    controller.destroy();
    controller = null;
    activity = null;
    myFragment = null;
    spyMyFragment = null;
}</pre> Our Unit Test will consist in testing that the Company BO is properly created when 

[onInflate()][2] method is called. To do so, we will populate a list of attributes that we will pass as parameter to onInflate by using [RoboAttributeSet][7]; <pre class="lang:java decode:true  ">@Test
public void shouldProperlyDecodeCustomAttributeSet() throws Exception {
    Company expected = new Company("Corp", 3);

    ArrayList&lt;Attribute&gt; attributes = new ArrayList&lt;Attribute&gt;();
    attributes.add(new Attribute("com.test.android:attr/company_name", "Corp", "com.test.android"));
    attributes.add(new Attribute("com.test.android:attr/company_age", String.valueOf(3), "com.test.android"));
    RoboAttributeSet attributeSet = new RoboAttributeSet(attributes, Robolectric.application.getResources(), null);

    myFragment.onInflate(activity, attributeSet, null);

    assertTrue(myFragment.company.equals(expected));
}</pre>

** # How about custom views ?** The previous steps remain identical (attrs.xml, extraction) for views apart from the method where the parsing occurs. Indeed, in Views, you can parse the custom attributes from the [constructor ][8]itself . **# Conclusion** Creating custom components often requires custom attributes to precisely initiate that component. What is really interesting here is the ability to properly test the parsing of the attributes using <a style="color: #f77e75;" href="http://robolectric.org/javadoc/org/robolectric/shadows/RoboAttributeSet.html">RoboAttributeSet</a> component.

 [1]: http://developer.android.com/reference/android/app/Fragment.html
 [2]: http://developer.android.com/reference/android/app/Fragment.html#onInflate(android.app.Activity,%20android.util.AttributeSet, android.os.Bundle)
 [3]: http://fr.wikipedia.org/wiki/Test_Driven_Development
 [4]: http://junit.org/
 [5]: https://code.google.com/p/mockito/
 [6]: http://robolectric.org/
 [7]: http://robolectric.org/javadoc/org/robolectric/shadows/RoboAttributeSet.html
 [8]: http://developer.android.com/reference/android/view/View.html#View(android.content.Context,%20android.util.AttributeSet)