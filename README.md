# AndroidBackgroundSync

The purpose of this tutorial is to create a way for your application to allow local modifications to your data that must be sent to a server, immediately if there is an internet connection or as soon as one is available.

First create a sync service. It will run in the background every time any activity or service launches a `Intent` for this service

```
public class SyncService extends IntentService {

    public SyncService() {
        super("SyncService");
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        syncEntities();
    }

    private void syncEntities() {
        // do your sync, call webservices, etc.
    }

}
```

Then create a broadcast receiver to listen for changes in the connectivity state of the wi-fi. This will guarantee that our sync will run as soon as wi-fi connectivity is available.

```
public class WifiBroadcastReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        ConnectivityManager conMan = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo netInfo = conMan.getActiveNetworkInfo();
        if (netInfo != null && netInfo.getType() == ConnectivityManager.TYPE_WIFI) {
            Intent intent2 = new Intent(context, SyncService.class);
            context.startService(intent2);
        } else {
            Log.d("WifiReceiver", "Don't have Wifi Connection");
        }
    }
}
```
Now register the BroadcastReceiver and the Service in `AndroidManifest.xml`, inside the `<application>` tag, before the activities.

```
<application>

<receiver android:name=".util.WifiBroadcastReceiver">
  <intent-filter>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>

<service
  android:name=".SyncService"
  android:exported="false" />
  
  ... activities declared here...
</application>
```

You can also try to sync after the user does something in an Activity. You just have to launch an `Intent` like the one in the BroadcastReceiver. It will run in the background as well.

