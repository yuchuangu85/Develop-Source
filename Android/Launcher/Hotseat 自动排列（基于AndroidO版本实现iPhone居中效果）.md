# Hotseat 自动排列（基于AndroidO版本实现iPhone居中效果）

思路：要实现图标自动排列：

第一步，要在每一个图标移除以后，将后面的图标向前面移动一个位置。并将该位置记录下来
第二步，要让hotseat左右间距保持一致。
第三考虑设备兼容

    遇到的一些问题：在做的过程中发现hotseat是一个celllayout，且行列数写死了，这就比较蛋疼了，尝试去动态修改行列数，却发现，图标之间的间距也会跟着改变，且在图标个数增加的时候，会导致两个图标重叠在一起的bug，无奈，最好只好通过改变margin的方式动态移动整个hotseat来实现居中

  主要代码如下

```java
在Launcher3\src\com\android\launcher3\dragndrop\DragDriver.java 的onTouchEvent()方法中添加如下方法的调用

	public void resetLayout(){
		int numHotseatIcons = mLauncher.getDeviceProfile().inv.numHotseatIcons;
        int j = 0;
		//移除或添加一个图标后，循环将图标重新排序
		for (int i = 0;i <= numHotseatIcons; i++) {
            View child = mLauncher.getHotseat().getLayout().getShortcutsAndWidgets().getChildAt(i,0);
			if(child != null){
	            CellLayout.LayoutParams clp = (CellLayout.LayoutParams) child.getLayoutParams();
				clp.cellX = j;
				j++;
			
				final ItemInfo info = (ItemInfo) child.getTag();
				//将重新排序存入数据库之中
				mLauncher.getModelWriter().modifyItemInDatabase(info, LauncherSettings.Favorites.CONTAINER_HOTSEAT, -1,
				                            clp.cellX, clp.cellY, 1, 1);
			}		
        }
		//考虑适配hotseat总数单双，居中显示hotseat
		FrameLayout.LayoutParams layoutParams = (FrameLayout.LayoutParams) mLauncher.getHotseat().getLayoutParams();
		if(j != numHotseatIcons){
			if (numHotseatIcons%2==0){
				mMargins = ((j == 1 ? numHotseatIcons - j - 1 : numHotseatIcons - j)*(mLauncher.getHotseat().getLayout().getMeasuredWidth() - (mLauncher.getDeviceProfile().hotseatLeftPadding * 2)))/(2*numHotseatIcons);
			}else{
				mMargins = ((numHotseatIcons - j)*(mLauncher.getHotseat().getLayout().getMeasuredWidth() - (mLauncher.getDeviceProfile().hotseatLeftPadding * 2)))/(2*numHotseatIcons);
			}
		}else{
			mMargins = 0;
		}
		if(j == 0)
		mMargins = 0;
		layoutParams.setMargins(mMargins,0,-mMargins,0);
		mLauncher.getHotseat().setLayoutParams(layoutParams);
		//重载hotseat
		mLauncher.getHotseat().getLayout().getShortcutsAndWidgets().requestLayout();
	}
```