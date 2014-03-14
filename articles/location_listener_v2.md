# Using location listener from Google Play Services


In this article I explain how to integrate the Location API in Google Play Services in your Android project.

First of all you need to set up the Google Play Services in your project. I'm using Android Studio with Gradle, here you have the steps that your have to follow to integrate play-services in your project:

* Install Google repository. Go to SDK Manager -> Extras -> install 'Google repository'

* In your build.gradle file you have to include:
```xml
	dependencies {
    	compile 'com.google.android.gms:play-services:4.2.+'
    }
```

Next, you have a snippet with the structure to make your Activity/Fragment location aware:
```java
public class LocationActivity extends Activity implements GooglePlayServicesClient.ConnectionCallbacks, 
	GooglePlayServicesClient.OnConnectionFailedListener, LocationListener {

	private final static int CONNECTION_FAILURE_RESOLUTION_REQUEST = 9000;
	// Milliseconds per second
	private static final int MILLISECONDS_PER_SECOND = 1000;
	// Update frequency in seconds
	public static final int UPDATE_INTERVAL_IN_SECONDS = 20;
	// Update frequency in milliseconds
	private static final long UPDATE_INTERVAL = MILLISECONDS_PER_SECOND * UPDATE_INTERVAL_IN_SECONDS;
	// The fastest update frequency, in seconds
	private static final int FASTEST_INTERVAL_IN_SECONDS = 15;
	// A fast frequency ceiling in milliseconds
	private static final long FASTEST_INTERVAL = MILLISECONDS_PER_SECOND * FASTEST_INTERVAL_IN_SECONDS;

	private LocationClient mLocationClient;
    private LocationRequest mLocationRequest;
    private Location mCurrentLocation;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		...
        // Location block
        mLocationClient = new LocationClient(this, this, this);
        // Create the LocationRequest object
        mLocationRequest = LocationRequest.create();
        // Use high accuracy
        mLocationRequest.setPriority(LocationRequest.PRIORITY_BALANCED_POWER_ACCURACY);
        // Set the update interval to 5 seconds
        mLocationRequest.setInterval(UPDATE_INTERVAL);
        // Set the fastest update interval to 1 second
        mLocationRequest.setFastestInterval(FASTEST_INTERVAL);
	}

	@Override
	public void onStart() {
		super.onStart();

		// Connect the client
		mLocationClient.connect();
	}
	
	@Override
    protected void onResume() {
        super.onResume();
        // Request location updates
        if (mLocationClient.isConnected()) {
            mLocationClient.requestLocationUpdates(mLocationRequest, this);
        }
    }
	
	@Override
    public void onPause() {
        super.onPause();
        if (mLocationClient.isConnected()) {
            mLocationClient.removeLocationUpdates(this);
        }
    }
    
	public void onStop() {
		super.onStop();

		mLocationClient.disconnect();
	}
	
	@Override
	public void onLocationChanged(Location location) {
		mCurrentLocation = location;
	}

	/*
	 * Called by Location Services when the request to connect the client finishes
	 * successfully. At this point, you can request the current location or start
	 * periodic updates
	 */
	@Override
	public void onConnected(Bundle dataBundle) {
		mCurrentLocation = mLocationClient.getLastLocation();
		mLocationClient.requestLocationUpdates(mLocationRequest, this);
	}

	/*
	 * Called by Location Services if the connection to the location client drops
	 * because of an error.
	 */
	@Override
	public void onDisconnected() {}
	
	 /*
	 * Called by Location Services if the attempt to Location Services fails.
	 */
    @Override
    public void onConnectionFailed(ConnectionResult connectionResult) {
		/*
		 * Google Play services can resolve some errors it detects. If the error has
		 * a resolution, try sending an Intent to start a Google Play services
		 * activity that can resolve error.
		 */
        if (connectionResult.hasResolution()) {
            try {
                // Start an Activity that tries to resolve the error
                connectionResult.startResolutionForResult(this, CONNECTION_FAILURE_RESOLUTION_REQUEST);
				/*
				 * Thrown if Google Play services canceled the original PendingIntent
				 */
            } catch (IntentSender.SendIntentException e) {
                // Log the error
                e.printStackTrace();
            }
        } else {
			/*
			 * If no resolution is available, display a dialog to the user with the
			 * error.
			 */

            // Get the error code
            int errorCode = connectionResult.getErrorCode();

            Dialog errorDialog = GooglePlayServicesUtil.getErrorDialog(errorCode, this, CONNECTION_FAILURE_RESOLUTION_REQUEST);
			errorDialog.show();
        }
    }
	
}

```

Finally, you have to include this permissions to your AndroidManifest.xml:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>

<application ... >
	<meta-data
            android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
</application>
```
