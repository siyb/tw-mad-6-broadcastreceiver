% MAD - Android 6: BroadcastReceiver
% Patrick Sturm
% 29.09.2016

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-mad-6-broadcastreceiver](https://github.com/siyb/tw-mad-6-broadcastreceiver)

# Agenda

## Agenda

* Introduction
* Theory
* Local Broadcasts

# ![Components](./component_broadcastreceiver.png)

# Introduction


## Introduction - 1 - Resources

* Javadoc: [http://developer.android.com/reference/android/content/BroadcastReceiver.html](http://developer.android.com/reference/android/content/BroadcastReceiver.html)

## Introduction - 2 - Basics

* BroadcastReceivers are Android components, they must be declared in the manifest, unless used in conjunction with LocalBroadcastManager
* As their name already states, BroadcastReceivers are used to receive some sort of broadcast
* They can receive broadcasts:
    * From within the application
    * From other applications
* -> Way of communicating between apps
* Works with intents (like Activities)

# Theory

## Theory - 1 - Manifest

```xml

<receiver
  android:exported="false“
  android:name=".broadcastreceiver.MyBroadcastReceiver">
</receiver>

```

* You can find a list of all supported attributes [here](https://developer.android.com/guide/topics/manifest/receiver-element.html)
* The receiver tag allows you to nest intent-filter, thus enabling your receiver to react to implicit intents (i.e. screen off/on)
* If you choose to set exported to true, your BroadcastReceiver can receive broadcasts from other applications as well, if set exported to false, the receiver only reacts to broadcasts issued by applications running under the same user account.

## Theory - 2 - Sending Broadcasts

* You can issue broadcasts by using the send(Sticky)Broadcast or send(Sticky)OrderedBroadcast respectively
* The sendBroadcast method will issue the broadcast asynchronously, meaning that the broadcast will be issued to all receivers at the same time (does not imply an order in which the BroadcastReceivers actually receive the broadcast).

## Theory - 3 - Type: Ordered

* The sendOrderBroadcast method will issue the broadcast to a receiver at a time (hence ordered)
  * This allows you to propagate the result from one receiver to another using the provided intent    
  * Abort the broadcast altogether, meaning that no other BroadcastReceiver will receive the broadcast (abortBroadcast())

## Theory - 4 - Type: Sticky

* The sticky variant of those methods will cause the start Intent to remain accessible once the broadcast is over
    * Your app needs to declare the BROADCAST_STICKY permission in AndroidManifest.xml
    * The state is still available after broadcast has been finished, thus allowing other components to access the data
    * Careful, the data Intent that sticks is a clone of the original Intent, meaning that changes to the Intent will not reflect on the sticky Intent!
    * Example: Intent batteryStatus = registerReceiver(null, filter);
        * Works because the battery status is a sticky BC, receiver can be null!
* Is used to cache broadcasts, updates can be issued, but your app can use a previous result to work with until a newer result is available

## Theory - 5 - Lifecycle and Lifecycle Implications

* There is no diagram available, but the BroadcastReceiver lifecycle is the most simple of all component lifecycles
* A BroadcastReceiver contains a single callback method, onReceive(...) only
* As soon as onReceive(...) exits, the BroadcastReceiver object is considered to be inactive, so don’t try the following:
    * Binding services
    * Running AsyncTasks
    * Spawning Dialogs

## Theory - 6 - Lifecycle and Lifecycle Implications cont.

* Even worse:
    * The process hosting the BroadcastReceiver typically does not see loads of user interaction (but it can!)
    * When onReceive(...) exists, the process that hosted the receiver will be killed forcefully, unless the process also hosts a long running Service / UI / etc
    * BroadcastReceivers can be inner classes
        * If they are started from somewhere else, imagine them to be public static final, since we do not have the context!
        * You CANNOT access any class members of the enclosing class, if they are not started from code (e.g. an Activity)

## Theory - 7 - Example

```java
public class Mbc 
  extends BroadcastReceiver { 
  private static final Logger LOGGER = LoggerFactory 
    .getLogger(Mbc.class); 

  public static final String ACTION_MYBCRACTION = 
    "com.sphericalelephant.Mbc.ACTION_MYBCRACTION"; 

  @Override 
  public void onReceive(Context context, Intent intent) { 
    // modifications to the intent will not
    // be visible in activity since we 
    // receive a copy of the original intent when calling
    // registerReceiver 
    LOGGER.debug("Broadcast received \\o/ "
      + intent.hashCode()); 
  } 
} 
```

# Local Broadcasts

## Local Broadcasts - 1 - Motivation

* Sometime it makes sense to issue local broadcasts
    * Are not accessible from outside your Application – hence you don‘t have to take extra security precautions 
    * Sometimes you only want to broadcast Intent within your application, there is no need to issue global broadcasts
* Use LocalBroadcastManager in order to issue local broadcasts
    * Does NOT support sticky broadcasts yet m( - (has been a few years now!)
    * Pseudo singleton, takes a Context, will utilize Application Context, thus not causing memory leaks
* If you use LocalBroadcastManager take the components lifecycle into account
    * Make sure to register / unregister the BroadcastReceivers within the appropriate lifecycle method (e.g. onResume(), onPause())

## Local Broadcasts - 2 - Motivation

```java
public class LocalBroadcastActivity extends Activity { 
  private MyBroadcastReceiver receiver = 
    new MyBroadcastReceiver(); 
  @Override protected void onResume() { 
    super.onResume(); 
    LocalBroadcastManager.getInstance(this)
      .registerReceiver(
        receiver = new MyBroadcastReceiver(), 
        new IntentFilter(
          "com.sphericalelephant.SOMEACTION")); 
  } 
  @Override protected void onPause() { 
    super.onPause(); 
    LocalBroadcastManager.getInstance(this)
      .unregisterReceiver(receiver); 
  } 
}
```

# Any Questions?