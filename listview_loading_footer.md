### Add a loading footer in ListView

This is a template to inclue a loading footer in a ListView inside a Fragment.

First, create a layout with the loading view:

```xml

	<?xmlversion="1.0"encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:gravity="center">

		<ProgressBar
			android:id="@+id/progressBar1"
			style="?android:attr/progressBarStyleLarge"
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"/>
 
	</LinearLayout>
```

Class structure:

```java
publicclassArticleListFragmentextendsListFragmentimplementsOnScrollListener{
 

…
 

@Override
public void onActivityCreated(Bundle savedInstanceState){
     super.onActivityCreated(savedInstanceState);
     ...
     // Pagination block
     // Load the loading footer view
     LayoutInflaterinf=LayoutInflater.from(getActivity());
     loadingFooter=inf.inflate(R.layout.view_loading_footer,null);
     getListView().addFooterView(loadingFooter);
     setListAdapter(adapter);
     getListView().setOnScrollListener(this);
     ...
}
 

	@Override
	public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
		if (firstVisibleItem + visibleItemCount >= (totalItemCount - 2) && moreItems && getListAdapter().getCount() > 0) {

			refreshData(false);
		}
	}

	@Override
	public void onScrollStateChanged(AbsListView arg0, int arg1) {

	}

``
