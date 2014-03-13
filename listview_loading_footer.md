### Add a loading footer in ListView

This is a template to inclue a loading footer in a ListView inside a Fragment.

First, create a layout with the loading view:

```xml
<?xmlversion="1.0"encoding="utf-8”?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android”
	android:layout_width=“match_parent”
	android:layout_height=“wrap_content”
	android:gravity=“center”]]>
 

<ProgressBar
	android:id="@+id/progressBar1”
	style="?android:attr/progressBarStyleLarge”
	android:layout_width=“wrap_content”
	android:layout_height="wrap_content”/>
 

</LinearLayout]]>
```
