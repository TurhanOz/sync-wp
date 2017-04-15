---
ID: 110
post_title: 'Unit Test Android : starting an activity'
author: turhan
post_date: 2014-09-16 23:00:40
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/unit-test-android-starting-an-activity/
published: true
dsq_thread_id:
  - "3050426938"
---
In this article, we will cover how to test "starting an Activity". As you should know - if you are an android developer - you can start an activity by either calling [startActivity(Intent) ][1]or [startActivityForResult(Intent, int).][2] The first method belongs to the [Context][3] objet while the latest belongs to the [Activity][4] object. As testing environment, we will use [Robolectric][5] with [Mockito][6]. For the sake of this study, we will create an ActivityLauncherController that has 2 methods to launch an Activity. <pre class="lang:java decode:true">public class ActivityLauncherController {

  public void startNextActivity(Context context) {
    Intent nextActivityIntent = new Intent(context, NextActivity.class);
    context.startActivity(nextActivityIntent);
  }

  public void startNextActivityForResult(Activity activity, int requestCode) {
    Intent nextActivityIntent = new Intent(activity, NextActivity.class);
    activity.startActivityForResult(nextActivityIntent , requestCode);
  }
}</pre>   We rely on the Android framework, so the idea here is not testing that the NextActivity instances is effectively created when calling startActivity(), but rather testing that the ActivityLauncherController has properly called the android API that will start that Activity. 

<pre class="lang:java decode:true">@RunWith(RobolectricTestRunner.class)
public class ActivityLauncherControllerTest {
  private Context context;
  private ActivityLauncherController activityLauncherController;

  @Before
  public void setUp() throws Exception {
    context = Robolectric.getShadowApplication().getApplicationContext();
    activityLauncherController = new ActivityLauncherController();
  }

  @Test
  public void shouldStartNextActivity() throws Exception {
    Context spyContext = spy(context);
    Intent expectedIntent = new Intent(spyContext, NextActivity.class);

    activityLauncherController.startNextActivity(spyContext);

    verify(spyContext).startActivity(expectedIntent);
  }

  @Test
  public void shouldStartNextActivityForResult() throws Exception {
    int requestCode = 1200;
    Activity activity = spy(Robolectric.buildActivity(Activity.class).create().start().visible().get());
    Intent expectedIntent = new Intent(activity, NextActivity.class);

    activityLauncherController.startNextActivityForResult(activity, requestCode);

    verify(activity).startActivityForResult(expectedIntent, requestCode);
  }
}</pre>   As you can see, to test the collaboration with the Context object on the first test, we need to spy a real Context instance so we can verify the proper startActivity() call on that context. On the second test, as our startNextActivityForResult() method requires an Activity instance, we can build that instance thanks to Robolectric.buildActivity() and then verify methods call on it. 

### Conclusion Starting an activity is very simple on android. Testing it is not that difficult as we are testing the collaboration with the framework and not the effective creation of the activity.

 [1]: http://developer.android.com/reference/android/content/Context.html#startActivity(android.content.Intent)
 [2]: http://developer.android.com/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int)
 [3]: http://developer.android.com/reference/android/content/Context.html
 [4]: http://developer.android.com/reference/android/app/Activity.html
 [5]: http://robolectric.org/
 [6]: https://code.google.com/p/mockito/