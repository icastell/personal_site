# Using Google Maps v2


```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/map_frame"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent" >

</FrameLayout>
```


```java
public class MyMapFragment extends SupportMapFragment {

	private static final float MIN_DISTANCE = 100.0f;

	protected SupportMapFragment mMapFragment;
	protected GoogleMap mMap;
	protected int mZoomPadding;
	private BitmapDescriptor mBitmapDescriptor;
	// Since we are consuming the event this is necessary to
    // manage closing openned markers before openning new ones
    private Marker mLastOpenned = null;
    private float mLastZoom;
    // Timer para no hacer muchas peticiones
    // mientras el usuario mueve el mapa
    private Handler mHandler = new Handler();
    
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
		// This is mandatory when use subclass a SupportMapFragment
		super.onCreateView(inflater,container,savedInstanceState);
       	...
	}

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

		// Map pin
		mBitmapDescriptor = BitmapDescriptorFactory.fromResource(R.drawable.pin);
        // Create the map view
        mMapFragment = SupportMapFragment.newInstance();
        // If we have the map embebed in a Activity we use
        // getSupportFragmentManager() instead getChildFragmentManager()
        FragmentTransaction transaction = getChildFragmentManager().beginTransaction();
        transaction.replace(R.id.map_frame, mMapFragment);
        transaction.commit();
        // Padding between the points and the screen borders when adjust the zoom level
        mZoomPadding = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 42, getResources().getDisplayMetrics());
    }
    
    @Override
    public void onResume() {
        super.onResume();

        setUpMapIfNeeded();

        if(mMap!=null){
            // Map is created
            // TODO: Draw the points
        }
    }
    
    /**
     * Draw in the map view a set of places
     * @param places
     */
    protected void drawPlacesInMap(ArrayList<Place> places){
        mMap.clear();
        ArrayList<LatLng> points = new ArrayList<LatLng>();
        for (Place place : places) {
            LatLng point = new LatLng(place.getLatitude(), place.getLongitude());
            mMap.addMarker(new MarkerOptions().position(point).title(place.getPlaceTitle()).snippet(place.getDescription()).icon(mBitmapDescriptor));
            points.add(point);
        }
        adjustZoom(points);
    }

    /**
     * Adjust a suitable zoom level to show all map points in the screen
     */
    protected void adjustZoom(ArrayList<LatLng> points) {
        if (points != null && points.size() > 0) {
            final LatLngBounds.Builder builder = new LatLngBounds.Builder();
            for (LatLng point : points) {
                builder.include(point);
            }
            CameraUpdate camera = CameraUpdateFactory.newLatLngBounds(builder.build(), mZoomPadding);
            try {
                mMap.animateCamera(camera);
            } catch (IllegalStateException e) {
                // To fix this: http://stackoverflow.com/a/13800112/1143172
                mMap.setOnCameraChangeListener(new GoogleMap.OnCameraChangeListener() {

                    @Override
                    public void onCameraChange(CameraPosition arg0) {
                        // Move camera.
                        mMap.moveCamera(CameraUpdateFactory.newLatLngBounds(builder.build(), mZoomPadding));
                        // Remove listener to prevent position reset on camera move.
                        mMap.setOnCameraChangeListener(null);
                    }
                });
                e.printStackTrace();
            }
        }
    }
    
    /**
	 * Sets up the map if it is possible to do so (i.e., the Google Play
	 * services APK is correctly installed) and the map has not already been
	 * instantiated.. This will ensure that we only ever call
	 * {@link #setUpMap()} once when {@link #mMap} is not null.
	 * <p>
	 * If it isn't installed {@link SupportMapFragment} (and
	 * {@link com.google.android.gms.maps.MapView MapView}) will show a prompt
	 * for the user to install/update the Google Play services APK on their
	 * device.
	 * <p>
	 * A user can return to this Activity after following the prompt and
	 * correctly installing/updating/enabling the Google Play services. Since
	 * the Activity may not have been completely destroyed during this process
	 * (it is likely that it would only be stopped or paused),
	 * {@link #onCreate(Bundle)} may not be called again so we should call this
	 * method in {@link #onResume()} to guarantee that it will be called.
	 */
	private void setUpMapIfNeeded() {
        // Do a null check to confirm that we have not already instantiated the
        // map.
        if (mMap == null) {
            // Getting Google Play availability status
            int status = GooglePlayServicesUtil.isGooglePlayServicesAvailable(getActivity());

            // Showing status
            if (status != ConnectionResult.SUCCESS) { // Google Play Services
                // are not
                // available
                int requestCode = 10;
                Dialog dialog = GooglePlayServicesUtil.getErrorDialog(status, getActivity(), requestCode);
                dialog.show();
            } else {
                // Try to obtain the map from the SupportMapFragment.
                // Check if we were successful in obtaining the map.
                mMap = mMapFragment.getMap();
                if (mMap != null) {
                    setUpMap();
                }
            }
        }
    }

	/**
	 * This is where we can add markers or lines, add listeners or move the
	 * camera. In this case, we just add a marker near Africa.
	 * <p>
	 * This should only be called once and when we are sure that {@link #mMap}
	 * is not null.
	 */
	private void setUpMap() {
		mMap.setMyLocationEnabled(true);
	}
}
```

Add this to your AndroidManifest.xml:
```xml
	<!-- GOOGLE MAPS -->
	<permission
        android:name="PACKAGE.permission.MAPS_RECEIVE"
        android:protectionLevel="signature" />
	<uses-feature
        android:glEsVersion="0x00020000"
        android:required="true" />
    <uses-feature
        android:name="android.hardware.location.gps"
        android:required="false" />
    <uses-feature
        android:name="android.hardware.location.network"
        android:required="false" />

    <uses-permission android:name="PACKAGE.permission.MAPS_RECEIVE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="com.google.android.providers.gsf.permission.READ_GSERVICES" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    
    ..
    
    <meta-data
            android:name="com.google.android.maps.v2.API_KEY"
            android:value="AIzaSyBEW1vPEIo7yNhOMRFqTLSYKLEuGws7_nw" />
    
    
```


### Maps features

Next, I show a couple of features that normally are going together with a map view.

* The snippet below show how we can obtain the screen radius:

```java
float[] diameter = new float[1];
// Get the visible rectangle
VisibleRegion region = map.getProjection().getVisibleRegion();
// Calculate the distance between northeast and southwest points
Location.distanceBetween(region.latLngBounds.northeast.latitude, region.latLngBounds.northeast.longitude, region.latLngBounds.southwest.latitude, region.latLngBounds.southwest.longitude, diameter);
float radius = diameter[0] / 2f;
```
* Map pin info window:

```java
private void setUpMap() {
	...
	// Pin click listener
	mMap.setOnInfoWindowClickListener(infoWindowClickListener);
	...
}
GoogleMap.OnInfoWindowClickListener infoWindowClickListener = new GoogleMap.OnInfoWindowClickListener() {
	 @Override
	public void onInfoWindowClick(Marker marker) {
		// TODO: Show details
	}
};
```
* Drag and query new points. If we want to get a new points when the user moves the screen we should do the next:

```java

	...
	private static final float MIN_DISTANCE = 100.0f;
	private static final long TIMER_GAP = 500;
	// Since we are consuming the event this is necessary to
    // manage closing openned markers before openning new ones
    private Marker mLastOpenned = null;
    private float mLastZoom;
    // Timer para no hacer muchas peticiones
    // mientras el usuario mueve el mapa
    private Handler mHandler = new Handler();
    private DragMapRunnable mDragMapRunnable;
    // Last position queried to the server
    private LatLng mLastLatLng;
    ...

private void setUpMap() {
	...
	// Listener to know when the user moves the screen
	mMap.setOnCameraChangeListener(cameraChangeListener);
	// Pin click listener
	mMap.setOnInfoWindowClickListener(infoWindowClickListener);
	// If the user tap on a pin the screen moves but we don'w want to recalculate de pins
	mMap.setOnMarkerClickListener(new GoogleMap.OnMarkerClickListener() {
	public boolean onMarkerClick(Marker marker) {
		// Check if there is an open info window
		if (mLastOpenned != null) {
 			// Close the info window
			mLastOpenned.hideInfoWindow();

			// Is the marker the same marker that was already open
			if (mLastOpenned.equals(marker)) {
				// Nullify the lastOpenned object
				mLastOpenned = null;
				// Return so that the info window isn't openned again
				return true;
			}
		}

		// Open the info window for the marker
		marker.showInfoWindow();
		// Re-assign the last openned such that we can close it later
		mLastOpenned = marker;

		// Event was handled by our code do not launch default behaviour.
		return true;
	}
	});
}

GoogleMap.OnCameraChangeListener cameraChangeListener = new GoogleMap.OnCameraChangeListener() {

        @Override
        public void onCameraChange(final CameraPosition cameraPosition) {

            float[] distance = new float[1];
            if (mLastLatLng != null) {
                Location.distanceBetween(cameraPosition.target.latitude, cameraPosition.target.longitude,
                        mLastLatLng.latitude, mLastLatLng.longitude, distance);
            }
            if (mLastLatLng == null || distance[0] > MIN_DISTANCE || cameraPosition.zoom != mLastZoom) {
                if (mDragMapRunnable != null) {
                    mHandler.removeCallbacks(mDragMapRunnable);
                }
                mDragMapRunnable = new DragMapRunnable(cameraPosition.target);
                mHandler.postDelayed(mDragMapRunnable, TIMER_GAP);
                mLastZoom = cameraPosition.zoom;
            }
        }
    };

    class DragMapRunnable implements Runnable {

        private LatLng cameraPosition;

        public DragMapRunnable(LatLng cameraPosition) {
            this.cameraPosition = cameraPosition;
        }

        @Override
        public void run() {
            // TODO: Query the server
        }
    }
``


