---
ID: 210
post_title: >
  AlarmManager Unit Test using Robolectric
  3.0
author: turhan
post_date: 2015-06-12 20:00:45
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/alarmmanager-unit-test-using-robolectric-3-0/
published: true
dsq_thread_id:
  - "3845708552"
---
<h2>Alarm Manager feature</h2>
<p style="text-align: justify;"><a href="http://developer.android.com/reference/android/app/AlarmManager.html">AlarmManager</a> is an Android component available since API level 1.</p>
<p style="text-align: justify;">I wanted to use this component in the following scenario :
- automatic and repeated triggers that ignite an operation n times a day
- any new scheduling should cancel previous ones (to avoid unconsistent multiple calls)
- when the device restarts (boot completed), the scheduling should be activated without running the app hosting the AlarmManager.
- the BootComplete operation is not enabled by default, but programatically.</p>

<h2>AlarmManager Unit Tests</h2>
<h3># Setting alarm</h3>
So I created an AlarmController that simply set repeated alarm and activate/desactivate scheduling on bootcomplete
<pre class="lang:java decode:true">public class AlarmController {
  
    public void setAlarm(Context context) {
        AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);

        Intent intent = new Intent(context, AlarmManagerBroadcastReceiver.class);
        PendingIntent alarmIntent = PendingIntent.getBroadcast(context, 0, intent, 0);

        alarmManager.cancel(alarmIntent);

        alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME,
                SystemClock.elapsedRealtime() + INTERVAL_ONE_HOUR,
                INTERVAL_ONE_HOUR, alarmIntent);
    }

    public void enableAlarmOnBootComplete(Context context) {
        //code on github
    }

    public void disableAlarmBootComplete(Context context) {
        //code on github
    }
}</pre>
<p style="text-align: justify;">The setAlarm() function simply triggers a broadcast at the given interval (here one hour).</p>
<p style="text-align: justify;">To test this with Robolectric 3.0, we have to use a <a href="https://github.com/robolectric/robolectric/blob/master/robolectric-shadows/shadows-core/src/main/resources/org/robolectric/shadows/ShadowAlarmManager.java.vm">shadowAlarmManager</a> that will store the parameters of the scheduled Alarm. We can thus simply test the interval set, the trigger time as well as the alarm type (here <a href="http://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME">elapsed_realtime</a>) of the ScheduledAlarm.</p>

<pre class="lang:java decode:true">@RunWith(RobolectricGradleTestRunner.class)
@Config(constants = BuildConfig.class, sdk = 21)
public class AlarmControllerTest {
    Context context;
    ShadowAlarmManager shadowAlarmManager;
    AlarmController sut;

    @Before
    public void setUp() throws Exception {
        context = RuntimeEnvironment.application.getApplicationContext();
        AlarmManager alarmManager = (AlarmManager) RuntimeEnvironment.application.getSystemService(Context.ALARM_SERVICE);
        shadowAlarmManager = shadowOf(alarmManager);

        sut = new AlarmController();
    }

    @Test
    public void shouldScheduleAlarmEveryHourStartingInOneHour() throws Exception {
        assertNull(shadowAlarmManager.getNextScheduledAlarm());

        sut.setAlarm(context);

        ShadowAlarmManager.ScheduledAlarm scheduledAlarm = shadowAlarmManager.getNextScheduledAlarm();
        assertEquals(AlarmController.INTERVAL_ONE_HOUR, scheduledAlarm.interval);
        assertEquals(SystemClock.elapsedRealtime() + AlarmController.INTERVAL_ONE_HOUR, scheduledAlarm.triggerAtTime);
        assertEquals(AlarmManager.ELAPSED_REALTIME, scheduledAlarm.type);
    }
}</pre>
<h3 style="text-align: justify;"># Ensuring one Alarm at a time</h3>
Let's say we call setAlarm() api multiple times (if called for instance during a lifecycle callback). Then, we have to ensure that only one alarm dealing with the same operation is set, avoiding thus any unconsistent and unnecessary mutliple wakes up.
<pre class="lang:java decode:true ">    @Test
    public void shouldCancelPreviousAlarmsAndKeepLatest() throws Exception {
        sut.setAlarm(context);
        sut.setAlarm(context);
        sut.setAlarm(context);

        assertEquals(1, shadowAlarmManager.getScheduledAlarms().size());
    }</pre>
<h3 style="text-align: justify;"># Checking Operation triggered by the Alarm</h3>
<p style="text-align: justify;">To test which <a href="http://developer.android.com/reference/android/app/PendingIntent.html">PendingIntent</a> operation is triggered by the Alarm, we have to use now a <a href="https://github.com/robolectric/robolectric/blob/master/robolectric-shadows/shadows-core/src/main/java/org/robolectric/shadows/ShadowPendingIntent.java">ShadowPendingIntent</a> which will give, once again, extra information, especially regarding the component.</p>

<pre class="lang:java decode:true  ">@Test
public void shouldTriggerBroadcastReceiverWhenTimeElapsed() throws Exception {
    Intent expectedIntent = new Intent(context, AlarmManagerBroadcastReceiver.class);

    sut.setAlarm(context);

    ShadowAlarmManager.ScheduledAlarm scheduledAlarm = shadowAlarmManager.getNextScheduledAlarm();
    ShadowPendingIntent shadowPendingIntent = shadowOf(scheduledAlarm.operation);
    assertTrue(shadowPendingIntent.isBroadcastIntent());
    assertEquals(1, shadowPendingIntent.getSavedIntents().length);
    assertEquals(expectedIntent.getComponent(), shadowPendingIntent.getSavedIntents()[0].getComponent());
}</pre>
<h3># Checking Operation triggered by the BroadcastReceiver</h3>
Our broadcast receiver has 2 goals : setting the alarm if BootCompleted and triggering a service (hosting the operation).
<pre class="lang:java decode:true ">public class AlarmManagerBroadcastReceiver extends BroadcastReceiver {
    private static final String ACTION_BOOT_COMPLETED = "android.intent.action.BOOT_COMPLETED";
    AlarmController alarmController;

    public AlarmManagerBroadcastReceiver() {
        alarmController = new AlarmController();
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        if (ACTION_BOOT_COMPLETED.equals(intent.getAction())) {
            alarmController.setAlarm(context);
        }

        Intent service = new Intent(context, SomeService.class);
        context.startService(service);
    }
}</pre>
To Unit Test that the service is properly started when broadcast is received, we have, once again, to shadow application so we can access to startedServices.
<pre class="lang:java decode:true">@RunWith(RobolectricGradleTestRunner.class)
@Config(constants = BuildConfig.class, sdk = 21)
public class AlarmManagerBroadcastReceiverTest {
    Context context;
    AlarmManagerBroadcastReceiver sut;

    @Before
    public void setUp() throws Exception {
        context = RuntimeEnvironment.application.getApplicationContext();
        sut = new AlarmManagerBroadcastReceiver();
        sut.alarmController = mock(AlarmController.class);
    }

    @Test
    public void onReceivingAnyIntentShouldStartService() throws Exception {
        Intent expectedService = new Intent(context, SomeService.class);

        sut.onReceive(context, new Intent());

        assertEquals(expectedService.getComponent(),
                shadowOf(RuntimeEnvironment.application).getNextStartedService().getComponent());
    }
}</pre>
<h3># Checking BroadcastReceiver will init Alarm on BootCompleted</h3>
To Unit Test that the Alarm is properly set onBootComplete, we simply verify, using mockito, the colloboration between the System Under Test (namely the BroadcastReceiver) and the AlarmController
<pre class="lang:java decode:true ">@RunWith(RobolectricGradleTestRunner.class)
@Config(constants = BuildConfig.class, sdk = 21)
public class AlarmManagerBroadcastReceiverTest {
    Context context;
    AlarmManagerBroadcastReceiver sut;
    
    //...

    @Test
    public void onReceivingAnyIntentShouldNotCollaborateWithAlarmController() throws Exception {
        sut.onReceive(context, new Intent());

        verify(sut.alarmController, never()).setAlarm(context);
    }

    @Test
    public void onBootCompleteShouldCollaborateWithAlarmController() throws Exception {
        sut.onReceive(context, new Intent("android.intent.action.BOOT_COMPLETED"));

        verify(sut.alarmController).setAlarm(context);
    }
}</pre>
<h3>#Activating / Desactivating Broadcast receiver on Demand</h3>
<p style="text-align: justify;">As you know, a Broadcast Receiver is a main component of Android Architecture and any receivers should be declared in the Android Manifest. However, let's suppose we want to activate our broadcast receiver only in a particular scenario  (after an in-app purchase for instance  or only during specific lifecycle callbacks), then how to proceed ?</p>

<pre class="lang:xhtml mark:2 decode:true">&lt;receiver android:name=".alarm.AlarmManagerBroadcastReceiver" 
        android:enabled="false"&gt;
    &lt;intent-filter&gt;
        &lt;action android:name="android.intent.action.BOOT_COMPLETED" /&gt;
    &lt;/intent-filter&gt;
&lt;/receiver&gt;</pre>
<p style="text-align: justify;">To do so, we have to declare the Broadcast receiver with the <a href="http://developer.android.com/reference/android/R.styleable.html#AndroidManifestReceiver_enabled">enable=false</a> flag. This gives us the control on programatically enabling/disabling the receiver. Then, we can use the Android PackageManager and set the state of our component (namely the receiver) on demand, as follow :</p>

<pre class="lang:java decode:true ">public class AlarmController {
  
    public void setAlarm(Context context) {
        //code on github
    }

    public void enableAlarmOnBootComplete(Context context) {
        ComponentName receiver = new ComponentName(context, AlarmManagerBroadcastReceiver.class);
        PackageManager pm = context.getPackageManager();

        pm.setComponentEnabledSetting(receiver,
                PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
                PackageManager.DONT_KILL_APP);
    }

    public void disableAlarmBootComplete(Context context) {
        ComponentName receiver = new ComponentName(context, AlarmManagerBroadcastReceiver.class);
        PackageManager pm = context.getPackageManager();

        pm.setComponentEnabledSetting(receiver,
                PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
                PackageManager.DONT_KILL_APP);
    }
}</pre>
<p style="text-align: justify;">To Unit Test this with Robolectric 3.0+, we rely on the <a href="https://github.com/robolectric/robolectric/blob/master/robolectric-resources/src/main/java/org/robolectric/res/builder/RobolectricPackageManager.java">RobolectricPackageManager</a> accessible through the <a href="https://github.com/robolectric/robolectric/blob/master/robolectric-resources/src/main/java/org/robolectric/RuntimeEnvironment.java">RuntimeEnvironment</a> object. This will give access to any component state, especially the one we are interested in (our receiver).</p>

<pre class="lang:java decode:true">    @Test
    public void broadcastReceiverShouldBeRegisteredWhenSet() throws Exception {
        ComponentName receiver = new ComponentName(context, AlarmManagerBroadcastReceiver.class);

        sut.enableAlarmOnBootComplete(context);

        assertEquals(PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
                RuntimeEnvironment.getRobolectricPackageManager().getComponentState(receiver).newState);
    }

    @Test
    public void broadcastReceiverShouldBeUnregisteredWhenUnSet() throws Exception {
        ComponentName receiver = new ComponentName(context, AlarmManagerBroadcastReceiver.class);

        sut.disableAlarmBootComplete(context);

        assertEquals(PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
                RuntimeEnvironment.getRobolectricPackageManager().getComponentState(receiver).newState);
    }</pre>
&nbsp;
<h2>Android Unit Test : source code on Github</h2>
I have create a new Open Source Project called <a href="https://github.com/TurhanOz/AndroidUnitTest">Android Unit Test</a> including this Alarm Manager testing. This project will gather interesting Android Unit Test scenario using Robolectric 3.0, Juni4.+, Mockito 1.9.+

Writing quality Unit Tests on Android requires sometimes quite struggling. This is why this open source project will help you save some time hopefully.