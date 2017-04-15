---
ID: 115
post_title: 'Unit test Android : testing Fragment&rsquo;s onSaveInstanceState'
author: turhan
post_date: 2014-09-17 22:25:22
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/unit-test-android-testing-fragments-onsaveinstancestate/
published: true
dsq_thread_id:
  - "3042461836"
---
On a <a href="http://turhanoz.com/saving-fragment-state-on-configuration-change/">previous article</a>, we saw how to retain some state of a custom Fragment when the latter is killed and recreated. Below a simple custom logical Fragment that saves a Parcelable into the state bundle and restore it.
<pre class="lang:default decode:true ">public class CustomFragment extends Fragment {
  Point point;

  @Override
  public void onSaveInstanceState(Bundle outState) {
    outState.putParcelable("point", point);
    super.onSaveInstanceState(outState);
  }

  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    if (savedInstanceState != null) {
      point = savedInstanceState.getParcelable("point");
    } else {
      point = new Point(0, 0);
    }
  return null;
  }
}</pre>
The specificity of the test here will be to simulate an activity recreation (as the lifecycle of a Fragment is higly tight to the
Activity's one) that has the CustomFragment as child.
As testing environment, we will use <a href="http://robolectric.org/">Robolectric</a> with <a href="https://code.google.com/p/mockito/">Mockito</a>.
<pre class="lang:java decode:true ">@RunWith(RobolectricTestRunner.class)
public class CustomFragmentTest {
  private static final String FRAGMENT_TAG = "customFragment";

  FragmentActivity activity;
  CustomFragment customFragment;

  @Before
  public void setUp() throws Exception {
    activity = Robolectric.buildActivity(FragmentActivity.class).create().start().resume().get();
    customFragment = new CustomFragment();
  }

  @Test
  @TargetApi(11)
  public void shouldRestoreFragmentWhenHostActivityIsRecreated() throws Exception {
    addMapFragment(activity, customFragment);
    Point expectedPoint = new Point(100, 200);
    customFragment.point = expectedPoint;

    activity.recreate();
    Fragment recreatedFragment = activity.getSupportFragmentManager().findFragmentByTag(FRAGMENT_TAG);

    assertNotNull(recreatedFragment);
    assertNotSame(customFragment, recreatedFragment);
    assertThat(((CustomFragment) recreatedFragment).point, is(expectedPoint));
  }

  private void addMapFragment(FragmentActivity activity, Fragment fragment) {
    FragmentManager fragmentManager = activity.getSupportFragmentManager();
    FragmentTransaction transaction = fragmentManager.beginTransaction();
    transaction.add(fragment, FRAGMENT_TAG);
    transaction.commit();
  }
}</pre>
&nbsp;

After having instanciated a CustomFragment as well as an Activity, we add that fragment to the activity.
There after, we modify the value of our parcelable (here a Point) that will be saved and restored through the Fragment lifecycle.
Then, we need to go through that life cycle, this is why we use the <a href="https://developer.android.com/reference/android/app/Activity.html#recreate()">recreate()</a> method of Activity that will create a new instance of that activity (note that this API is level 11, this is why we annotate our test with @TargetApi(11)), as well as a new instance of our Fragment.
So we get a reference of this new recreatedFragment and simply test that the initial customFragment is a different instance of the newly recreated one and of course, we test that the parcelable has the proper value (the one set before the saving).
<h3>Conclusion :</h3>
Instead of testing that the Parcelable is saved into the outState Bundle of the onSaveInstanceState() method and properly restored on the onCreateView() method, we simply verify that the Parcelable is correctly handled through the whole lifeCycle. This gives us the elegancy of ignoring where the savedInstanceState is restored (<a href="http://turhanoz.com/saving-fragment-state-on-configuration-change/">as there are various methods that could do that</a>) and test that at a higher level.