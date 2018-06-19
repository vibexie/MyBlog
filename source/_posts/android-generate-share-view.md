---
title: Android后台生成分享图片
date: 2016-08-06 00:47:16
tags: android
---
现在有很多App在社会化分享的时候，都会分享一张图片，并且图片中的内容是根据相关的数据绘制生成。如果你在开发中遇到这种需求，你需要先考虑下面两个问题:

1. 需要分享的图片刚刚好是当前Activity界面或者是界面中的一部分，怎么处理好？
恭喜你，这个就是个截图而已，就是把当前Activity截图下来得到一个bitmap。。。

2. 需要分享的图片不是当前Activity界面，怎么办？
那就是需要在后台去生成张图片了，这就是我们要讨论的问题。

<!-- more -->
好的，既然是要在后台生成图片，我们的思路就是，最好有一个layout，我们用xml先把布局写好，在通过java代码把layout中的控件都填上我们的数据，这应该是最好的实现方式了，如果你想完全用Java代码，把View绘制出来，那就是要吃力点。
然而，就算是已经把layout inflate进来了，把控件数据都填上了，但是怎么截图呢？或许你和我一样，第一次开发这样的东西，怎么截都截不到view的bitmap！这里不能像截Activity图一样哦，因为没有显示出来的View，是截取不到的。没关系，办法还是有的，请看下面代码...
``` java
public class ShareView {
    //分享页面的View
    private View mView;
	private Activity mActivity;
	
	public ShareView(Activity activity) {
		// TODO Auto-generated constructor stub
		this.mActivity = activity;
		//inflate XML布局
		this.mView = LayoutInflater.from(activity).inflate(R.layout.view_share, null);
	}
	
	/**
	 * 设置mView中的数据
	 */
	public void alreadyToGenerate() {
		TextView tv = (TextView) mView.findViewById(R.id.view_share_tv);
		tvSuggest.setText("哈哈");
	}
	
	/**
	 * 生成图片核心代码
	 */
	public Bitmap generateBitmap() {
		mView.setDrawingCacheEnabled(true);
		//图片的宽度为屏幕宽度，高度为wrap_content
		mView.measure(MeasureSpec.makeMeasureSpec(mActivity.getResources().getDisplayMetrics().widthPixels, MeasureSpec.EXACTLY), MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED));     
		//放置mView
		mView.layout(0, 0, mView.getMeasuredWidth(), mView.getMeasuredHeight());     
		mView.buildDrawingCache();
		Bitmap bitmap = mView.getDrawingCache();
		return bitmap;
	}
}
```

