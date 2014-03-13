### Add a loading footer in ListView

This is a template to include a loading footer in a ListView inside a Fragment.

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

Following, you have a class structure:

```java
public class ArticleListFragment extends ListFragment implements OnScrollListener {

	...
	protected View mLoadingFooter;
	protected boolean mFirstRefresh = true, mMoreItems = false;
	...
	
	@Override
	public void onActivityCreated(Bundle savedInstanceState) {
		super.onActivityCreated(savedInstanceState);
		...
		// Pagination block
		// Load the loading footer view
		LayoutInflater inf = LayoutInflater.from(getActivity());
		mLoadingFooter = inf.inflate(R.layout.view_loading_footer, null);
		getListView().addFooterView(loadingFooter);
		setListAdapter(adapter);
		getListView().setOnScrollListener(this);
		
		mFirstRefresh = true;
		...
	}
 

	@Override
	public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
		if (firstVisibleItem + visibleItemCount >= (totalItemCount - 2) 
		&& mMoreItems && getListAdapter().getCount() > 0) {
			refreshData();
		}
	}

	private void refreshData() {
		// To not load while we are downloading
		mMoreItems = false;
		// TODO: Refresh data
	}

	@Override
	public void onScrollStateChanged(AbsListView arg0, int arg1) {

	}
	
	public void onFinishLoadData(Response response) {
		// Important! This has to be executed in UI thread 
		if(response.getStatus == SUCCESS) {
			int numItems = response.getNumItems();
			mMoreItems = numItems == LIMIT;
			if (mFirstRefresh) {
				if(mListView.getFooterViewsCount()>0){
					mListView.removeFooterView(mLoadingFooter);
				}
				mListView.addFooterView(mLoadingFooter);
			}
			mFirstRefresh = false;
			// Delete the footer if no more items
			if (!mMoreItems && mListView.getFooterViewsCount() > 0) {
				mListView.removeFooterView(mLoadingFooter);
			}
		} else {
			mMoreItems = false;
			if (mListView.getFooterViewsCount()>0) {
				mListView.removeFooterView(mLoadingFooter);
			}
		}
	}
}

```
