---
title: "A Simple Primer On Android Priority Queue And Greenrobot Eventbus"
excerpt: "Android asynchronous tasks without the difficulty"
categories: kotlin java android asynchronous
header:
  image: /assets/images/greenbus.jpg
---

Dealing with asynchronous tasks in Android can be a headache. I have been introduced to two open source libraries that make this all the easier. I will attempt to explain the basics here to get you started with them. I will show an Android Studio boiler plate application that creates a simple job, that then returns to the UI thread, where we display a toast.

Android Priority Job Queue is an implementation of a Job Queue specifically written for Android to easily schedule jobs (tasks) that run in the background. I will use this to manage the execution of the job.

EventBus enables central communication to decoupled classes using the publisher/subscriber pattern. I will use this to communicate the result of the finished job back to the application, the calling Activity and/or a POJO.

Create an app, then add the libraries via gradle (up to date versions at the time of writing)

```gradle
dependencies {
... 
...  
    //Add latest jobqueue and greenrobot eventbus
    compile 'com.birbit:android-priority-jobqueue:2.0.1'
    compile 'org.greenrobot:eventbus:3.0.0'
}
```

I am going to initialise both libraries from a application object, so I need to add one of those to the app.

```java
public class MyApplication extends Application {
    public JobManager jobManager;
    public EventBus eventBus;

    @Override
    public void onCreate() {
        super.onCreate();

        //Setup JobManager
        Configuration.Builder builder = new Configuration.Builder(this);
        jobManager = new JobManager(builder.build());

        //Setup EventBus
        eventBus = EventBus.getDefault();
    }

}
```

then I need to update my manifest to make it aware of the application object

```xml
<application
    android:name=".MyApplication"
```

Now I will create a simple job.

```java
public class DelayJob extends Job {
    DelayJob() { super(new Params(1));
    }

    @Override
    public void onAdded() {
    }

    @Override
    public void onRun() throws Throwable {
        //Simple job that takes time
        Thread.sleep(5000);

        MyApplication application = (MyApplication) getApplicationContext();
        //Send event back to subscribers of MyEventMessage
        application.eventBus.post(new MyEventMessage("Hello world!"));
    }

    @Override
    protected void onCancel(int i, @Nullable Throwable throwable) {
    }

    @Override
    protected RetryConstraint shouldReRunOnThrowable(@NonNull Throwable throwable, int i, int i1) {
        return null;
    }
}
```

Here I am just adding to the `onRun()` method. It’s a simple job that sleeps for 5 seconds and then uses the EventBus to post an event back. The event just a class, and in my case it’s a Kotlin data class which does a lot of the boiler plate code for you.

```kotlin
data class MyEventMessage ( var message : String );
```

To run the event, just add it to the jobmanager instance you have created.

```java
//Queue simple job
final MyApplication application = (MyApplication) getApplicationContext();
application.jobManager.addJobInBackground(new DelayJob());
```

The event just a class, and in my case it’s a Kotlin data class which does a lot of the boiler plate code for you. You could use a simple Java clas, or something more complicated.

```kotlin
data class MyEventMessage ( var message : String );
```

Now I need to subscribe to this event. This can be done in any class, providing you register with EventBus. Here I will show this being done in an activity

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

...

        //Register for events
        EventBus.getDefault().register(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //Don't forget to unregister from EventBus
        EventBus.getDefault().unregister(this);
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEvent(MyEventMessage message) {
        new MyToast().show(this, "Activity:" + message.getMessage());
    }
}
```

The key to receiving the event is the annotation Subscribe and the functions parameter type. In this case my function onEvent will receive all MyEventMessage classes that are posted to EventBus. The ThreadMode allows me to specify what thread the function will be called in. Here I am asking for my function to be called in the UI thread. The function name is not important.

For more information I have a git repo with this working example. It also shows receiving the event in the Application object and creating a POJO that also receives the same event after creation. Please visit the respective EventBus and Android Priority Job Queue for more in-depth help.

[Original Post](https://medium.com/@ubikspace/a-simple-primer-on-android-priority-queue-and-greenrobot-eventbus-e77e246ba3d2)