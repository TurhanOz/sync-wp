---
ID: 129
post_title: Mocking Bound Services on Android
author: turhan
post_date: 2014-11-09 20:00:59
post_excerpt: ""
layout: post
permalink: >
  http://turhanoz.com/mocking-bound-services-on-android/
published: true
dsq_thread_id:
  - "3211650513"
---
Let's suppose we have a Fragment that we want to unit test.
However, let's also suppose that this fragment binds to/unbinds from a service through its lifecycle callback (<a href="http://developer.android.com/reference/android/app/Fragment.html#onStart()">onStart()</a>, <a href="http://developer.android.com/reference/android/app/Fragment.html#onStop()">onStop()</a>).

The main advantage of <a href="(http://developer.android.com/guide/components/bound-services.html">Bound Service</a>  is that the latter lives only if at least one client is bound to it. By binding to/unbinding from a Service through a Fragment or Activity's lifecycle, we are in full control of the existence of that Service and triggers it only when the user needs it.

To use a bound service, you mainly need 2 classes : an <a href="http://developer.android.com/reference/android/app/Service.html">Android Service</a> and a <a href="http://developer.android.com/reference/android/content/ServiceConnection.html">ServiceConnection</a> that monitors the state of the service connection (Service connected/disconnected).

Below a very simple Service and ServiceConnection :
<pre class="lang:java decode:true " title="LocalService">public class LocalService extends Service{
    final IBinder localBinder = new LocalBinder();

    @Override
    public IBinder onBind(Intent intent) {
        return localBinder;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        doExpensiveTask();
    }

    private void doExpensiveTask(){
       // for instance, expensive threaded task from network, io &amp; so ...
    }

    public class LocalBinder extends Binder {
        LocalService getService() {
            return LocalService.this;
        }
    }
}</pre>
&nbsp;
<pre class="lang:java decode:true" title="LocalServiceConnector">public class LocalServiceConnector implements ServiceConnection {
    private Context context;
    private LocalService localService;
    private boolean isServiceBound;

    public LocalServiceConnector(Context context) {
        this.context = context;
    }

    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        LocalService.LocalBinder binder = (LocalService.LocalBinder) service;
        localService = binder.getService();
        isServiceBound = true;
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        isServiceBound = false;
    }

    public void bindToService() {
        Intent intent = new Intent(context, LocalService.class);
        context.bindService(intent, this, Context.BIND_AUTO_CREATE);
    }

    public void unbindFromService() {
        if (isServiceBound) {
            context.unbindService(this);
        }
    }
}</pre>
As you can see, as soon as the service is created, an expensive task is triggered within the <a href="http://developer.android.com/reference/android/app/Service.html#onCreate()">onCreate()</a> callback.

To bind to the service, we will make proper use of the ServiceConnector within our Fragment's life cycle :
<pre class="lang:java decode:true " title="MyFragment">public class MyFragment extends Fragment {
    LocalServiceConnector localServiceConnector;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        initLocalServiceConnector();
        return inflater.inflate(R.layout.myfragment, container, false);
    }

    private void initLocalServiceConnector() {
        localServiceConnector = new LocalServiceConnector(getActivity());
    }

    @Override
    public void onStart() {
        super.onStart();
        localServiceConnector.bindToService();
    }

    @Override
    public void onStop() {
        super.onStop();
       localServiceConnector.unbindFromService();
    }
}</pre>
<h2></h2>
<h2>The Unit Test</h2>
As we have already seen, as soon as the service is created, an expensive task is triggered within the <a href="http://developer.android.com/reference/android/app/Service.html#onCreate()">onCreate()</a> callback. This could lead to pollute and add uncertainty to the result of our unit test (such as the IBinder being null within the <a href="http://developer.android.com/reference/android/content/ServiceConnection.html#onServiceConnected(android.content.ComponentName,%20android.os.IBinder)">onServiceConnected()</a> callback). Just to remind you, <strong><span style="text-decoration: underline;">a good unit test should be consistent</span></strong>, which means that it should always return the same result for the same test.

So in order not to be disrupted by the Service, we are goind to mock not only the Service, but also stub the IBinder. Thus, anytime a client (the fragment in our case) wants to bind to the Service, it will receive a stub IBinder that will return a mock Service.
Let's have a look to the Unit Test (we will use Robolectric and Mockito to do so ) and specifically to the <strong>mockBoundLocalService()</strong> method:
<pre class="lang:java decode:true">@RunWith(RobolectricTestRunner.class)
public class MyFragmentTest {
    FragmentActivity activity;
    ActivityController controller;
    MyFragment myFragment;

    @After
    public void tearDown() throws Exception {
        controller.destroy();
        controller = null;
        activity = null;
        myFragment = null;
    }

    @Before
    public void setUp() throws Exception {
        activity = createActivityControllerAndHostActivity();
        myFragment = new MyFragment();
        mockBoundLocalService();
    }

    private FragmentActivity createActivityControllerAndHostActivity() {
        controller = Robolectric.buildActivity(FragmentActivity.class);
        return (FragmentActivity) controller.create().start().resume().get();
    }

    private void mockBoundLocalService(){
        LocalService.LocalBinder stubBinder = mock(LocalService.LocalBinder.class);
        when(stubBinder.getService()).thenReturn(mock(LocalService.class));
        shadowOf(Robolectric.application).setComponentNameAndServiceForBindService(new ComponentName("com.company.project","LocalService"), stubBinder);
    }

    @Test
    public void shouldProperlyInit() throws Exception {
        addMyFragment(activity, myFragment);
        //Do whatever test of your fragment here
    }

    private void addMyFragment(FragmentActivity activity, Fragment fragment) {
        FragmentManager fragmentManager = activity.getSupportFragmentManager();
        FragmentTransaction transaction = fragmentManager.beginTransaction();
        transaction.add(fragment, "TAG");
        transaction.commit();
    }
}</pre>
The Robolectric's ShadowApplication object has a method called <a href="http://robolectric.org/javadoc/org/robolectric/shadows/ShadowApplication.html#setComponentNameAndServiceForBindService(android.content.ComponentName,%20android.os.IBinder)">setComponentNameAndServiceForBindService()</a> that allows to set a IBinder for the given Service <a href="http://developer.android.com/reference/android/content/ComponentName.html">ComponentName</a>. Using this setter, we are able to pass a stub IBinder within the <a href="http://developer.android.com/reference/android/content/ServiceConnection.html#onServiceConnected(android.content.ComponentName,%20android.os.IBinder)">onServiceConnected()</a> callback whenever a client tries to bind to our service.

&nbsp;