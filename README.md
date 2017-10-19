带有标签的     流布局的增删demo:

项目中经常遇到这样的例子：界面上很多标签，共用户选择，用户可以自行添加，自行删除。     随手写了一个：

效果图：



核心源码：

自定义ViewGroup：
FlowLayout.java
package com.ac.flowlayout_danxuan;

import java.util.ArrayList;
import java.util.List;

import android.content.Context;
import android.util.AttributeSet;
import android.view.View;
import android.view.ViewGroup;

/**
 * com.ac.flowlayout_danxuan
 * @Email zhaoq_hero@163.com
 * @author zhaoQiang : 2016-2-23
 */
public class FlowLayout extends ViewGroup {
	
	private static final String TAG = "FlowLayout";
	
	public FlowLayout(Context context, AttributeSet attrs) {
		super(context, attrs);
	}
	
	@Override
	protected ViewGroup.LayoutParams generateLayoutParams(ViewGroup.LayoutParams p) {
		return new MarginLayoutParams(p);
	}
	
	@Override
	public ViewGroup.LayoutParams generateLayoutParams(AttributeSet attrs) {
		return new MarginLayoutParams(getContext(), attrs);
	}
	
	@Override
	protected ViewGroup.LayoutParams generateDefaultLayoutParams() {
		return new MarginLayoutParams(LayoutParams.MATCH_PARENT,
				LayoutParams.MATCH_PARENT);
	}
	
	/**
	 * 负责设置子控件的测量模式和大小 根据所有子控件设置自己的宽和高
	 */
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		super.onMeasure(widthMeasureSpec, heightMeasureSpec);
		// 获得它的父容器为它设置的测量模式和大小
		int sizeWidth = MeasureSpec.getSize(widthMeasureSpec);
		int sizeHeight = MeasureSpec.getSize(heightMeasureSpec);
		int modeWidth = MeasureSpec.getMode(widthMeasureSpec);
		int modeHeight = MeasureSpec.getMode(heightMeasureSpec);
		
//		Log.e(TAG, sizeWidth + "," + sizeHeight);
		
		// 如果是warp_content情况下，记录宽和高
		int width = 0;
		int height = 0;
		/**
		 * 记录每一行的宽度，width不断取最大宽度
		 */
		int lineWidth = 0;
		/**
		 * 每一行的高度，累加至height
		 */
		int lineHeight = 0;
		
		int cCount = getChildCount();
		
		// 遍历每个子元素
		for (int i = 0; i < cCount; i++) {
			View child = getChildAt(i);
			// 测量每一个child的宽和高
			measureChild(child, widthMeasureSpec, heightMeasureSpec);
			// 得到child的lp
			MarginLayoutParams lp = (MarginLayoutParams) child
					.getLayoutParams();
			// 当前子空间实际占据的宽度
			int childWidth = child.getMeasuredWidth() + lp.leftMargin
					+ lp.rightMargin;
			// 当前子空间实际占据的高度
			int childHeight = child.getMeasuredHeight() + lp.topMargin
					+ lp.bottomMargin;
			/**
			 * 如果加入当前child，则超出最大宽度，则的到目前最大宽度给width，类加height 然后开启新行
			 */
			if (lineWidth + childWidth > sizeWidth) {
				width = Math.max(lineWidth, childWidth);// 取最大的
				lineWidth = childWidth; // 重新开启新行，开始记录
				// 叠加当前高度，
				height += lineHeight;
				// 开启记录下一行的高度
				lineHeight = childHeight;
			} else {
				// 否则累加值lineWidth,lineHeight取最大高度
				lineWidth += childWidth;
				lineHeight = Math.max(lineHeight, childHeight);
			}
			// 如果是最后一个，则将当前记录的最大宽度和当前lineWidth做比较
			if (i == cCount - 1) {
				width = Math.max(width, lineWidth);
				height += lineHeight;
			}

		}
		setMeasuredDimension((modeWidth == MeasureSpec.EXACTLY) ? sizeWidth : width,
				(modeHeight == MeasureSpec.EXACTLY) ? sizeHeight : height);
	}
	
	/**
	 * 存储所有的View，按行记录
	 */
	private List<List<View>> mAllViews = new ArrayList<List<View>>();
	
	public List<List<View>> getmAllViews() {
		return mAllViews;
	}

	public void setmAllViews(List<List<View>> mAllViews) {
		this.mAllViews = mAllViews;
	}

	/**
	 * 记录每一行的最大高度
	 */
	private List<Integer> mLineHeight = new ArrayList<Integer>();
	
	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
		mAllViews.clear();
		mLineHeight.clear();

		int width = getWidth();

		int lineWidth = 0;
		int lineHeight = 0;
		// 存储每一行所有的childView
		List<View> lineViews = new ArrayList<View>();
		int cCount = getChildCount();
		// 遍历所有的孩子
		for (int i = 0; i < cCount; i++) {
			View child = getChildAt(i);
			MarginLayoutParams lp = (MarginLayoutParams) child
					.getLayoutParams();
			int childWidth = child.getMeasuredWidth();
			int childHeight = child.getMeasuredHeight();

			// 如果已经需要换行
			if (childWidth + lp.leftMargin + lp.rightMargin + lineWidth > width) {
				// 记录这一行所有的View以及最大高度
				mLineHeight.add(lineHeight);
				// 将当前行的childView保存，然后开启新的ArrayList保存下一行的childView
				mAllViews.add(lineViews);
				lineWidth = 0;// 重置行宽
				lineViews = new ArrayList<View>();
			}
			/**
			 * 如果不需要换行，则累加
			 */
			lineWidth += childWidth + lp.leftMargin + lp.rightMargin;
			lineHeight = Math.max(lineHeight, childHeight + lp.topMargin + lp.bottomMargin);
			lineViews.add(child);
		}
		// 记录最后一行
		mLineHeight.add(lineHeight);
		mAllViews.add(lineViews);

		int left = 0;
		int top = 0;
		// 得到总行数
		int lineNums = mAllViews.size();
		for (int i = 0; i < lineNums; i++) {
			// 每一行的所有的views
			lineViews = mAllViews.get(i);
			// 当前行的最大高度
			lineHeight = mLineHeight.get(i);

//			Log.e(TAG, "第" + i + "行 ：" + lineViews.size() + " , " + lineViews);
//			Log.e(TAG, "第" + i + "行， ：" + lineHeight);

			// 遍历当前行所有的View
			for (int j = 0; j < lineViews.size(); j++) {
				View child = lineViews.get(j);
				if (child.getVisibility() == View.GONE) {
					continue;
				}
				MarginLayoutParams lp = (MarginLayoutParams) child
						.getLayoutParams();

				//计算childView的left,top,right,bottom
				int lc = left + lp.leftMargin;
				int tc = top + lp.topMargin;
				int rc =lc + child.getMeasuredWidth();
				int bc = tc + child.getMeasuredHeight();

//				Log.e(TAG, child + " , l = " + lc + " , t = " + t + " , r ="
//						+ rc + " , b = " + bc);

				child.layout(lc, tc, rc, bc);
				
				left += child.getMeasuredWidth() + lp.rightMargin
						+ lp.leftMargin;
			}
			left = 0;
			top += lineHeight;
		}

	}
}

TagItm.java
package com.ac.flowlayout_danxuan;

import android.widget.TextView;

/**
 * com.ac.flowlayout_danxuan
 * @Email zhaoq_hero@163.com
 * @author zhaoQiang : 2016-2-23
 */
public class TagItem {
	public String tagText;  //标签上的文本信息：
	public boolean tagCustomEdit;
	public int idx;
	public TextView mView;
}

MainActivity.java
package com.ac.flowlayout_danxuan;

import java.util.ArrayList;

import android.app.Activity;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

/**
 * com.ac.flowlayout_danxuan
 * @Email zhaoq_hero@163.com
 * @author zhaoQiang : 2016-2-23
 */
public class MainActivity extends Activity {
	
	FlowLayout mTagLayout;
	private ArrayList<TagItem> mAddTags = new ArrayList<TagItem>();

	private EditText inputLabel;
	private Button btnSure;
	
	// 存放标签数据的数组
	String[] mTextStr = { "有点所", "有的", "控", "件都往左飘", "的感觉", "第一行满了", "往第二行飘" };
	
	ArrayList<String>  list = new ArrayList<String>();
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        inputLabel = (EditText) LayoutInflater.from(this).inflate(R.layout.my_edit_text, null);
        
        btnSure = (Button) findViewById(R.id.btn_sure);
        
        mTagLayout = (FlowLayout) findViewById(R.id.tag_layout);
        
        initList();
        
    	initLayout(list);
    	
    	initBtnListener();
    }
    
	private void initList() {
    	for(int i=0;i<mTextStr.length;i++){
    		list.add(mTextStr[i]);
    	}
	}

	private void initBtnListener() {
    	/**
         * 初始化  单击事件：
         */
    	btnSure.setOnClickListener(new OnClickListener() {
    			
    		@Override
    		public void onClick(View v) {

    			String label = inputLabel.getText().toString().trim();
    			
    			String[] newStr = new String[mTagLayout.getChildCount()];
    			
    			/**
    			 * 获取  子view的数量   并添加进去
    			 */
    			if(label!=null&&!label.equals("")){
    				for(int m = 0;m < mTagLayout.getChildCount()-1;m++){
    					newStr[m] =((TextView)mTagLayout.getChildAt(m).
    							findViewById(R.id.text)).getText().toString();//根据  当前   位置查找到 当前    textView中标签  内容
    				}
    				list.add(label);
    				initLayout(list);
    				inputLabel.setText("");
    			}
    		}
    	});
	}

    
	private void initLayout(final ArrayList<String> arr) {
		
		mTagLayout.removeAllViewsInLayout();
		/**
		 * 创建 textView数组
		 */
		final TextView[] textViews = new TextView[arr.size()];
	    final TextView[] icons = new TextView[arr.size()];
		
		for (int i = 0; i < arr.size(); i++) {
			
			final int pos = i;
			
			final View view = (View) LayoutInflater.from(MainActivity.this).inflate(R.layout.text_view, mTagLayout, false);
			
			final TextView text = (TextView) view.findViewById(R.id.text);  //查找  到当前     textView
			final TextView icon = (TextView) view.findViewById(R.id.delete_icon);  //查找  到当前  删除小图标

			// 将     已有标签设置成      可选标签
			text.setText(list.get(i));
			/**
			 * 将当前  textView  赋值给    textView数组
			 */
			textViews[i] = text;
			icons[i] = icon;
			
			//设置    单击事件：
			icon.setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {
					//遍历  图标  删除 当前  被点击项
					for(int j = 0; j < icons.length;j++){
						if(icon.equals(icons[j])){  //获取   当前  点击删除图标的位置：
							mTagLayout.removeViewAt(j);  
							list.remove(j);
							initLayout(list);
						}
					}
				}
			});
			
			text.setOnClickListener(new OnClickListener() {
				
				@Override
				public void onClick(View v) {
					
					text.setActivated(!text.isActivated()); // true是激活的 
					
					if (text.isActivated()) {
						boolean bResult = doAddText(list.get(pos), false, pos);
						text.setActivated(bResult);
						//遍历   数据    将图标设置为可见：
						for(int j = 0;j< textViews.length;j++){
							if(text.equals(textViews[j])){//非当前  textView
								icons[j].setVisibility(View.VISIBLE);
							}
						}
					}else{
						for(int j = 0;j< textViews.length;j++){
							icons[j].setVisibility(View.GONE);
						}
					}
					
					/**
					 * 遍历  textView  满足   已经被选中     并且不是   当前对象的textView   则置为  不选
					 */
					for(int j = 0;j< textViews.length;j++){
						if(!text.equals(textViews[j])){//非当前  textView
							textViews[j].setActivated(false); // true是激活的 
							icons[j].setVisibility(View.GONE);
						}
					}
				}
			});
			
			mTagLayout.addView(view);
		}
		
		mTagLayout.addView(inputLabel);
	}
	
	// 标签索引文本
	protected int idxTextTag(String text) {
		int mTagCnt = mAddTags.size(); // 添加标签的条数
		for (int i = 0; i < mTagCnt; i++) {
			TagItem item = mAddTags.get(i);
			if (text.equals(item.tagText)) {
				return i;
			}
		}
		return -1;
	}
	
	// 标签添加文本状态
	private boolean doAddText(final String str, boolean bCustom, int idx) {
		int tempIdx = idxTextTag(str);
		if (tempIdx >= 0) {
			TagItem item = mAddTags.get(tempIdx);
			item.tagCustomEdit = false;
			item.idx = tempIdx;
			return true;
		}
		int tagCnt = mAddTags.size(); // 添加标签的条数
		TagItem item = new TagItem();
		item.tagText = str;
		item.tagCustomEdit = bCustom;
		item.idx = idx;
		mAddTags.add(item);
		tagCnt++;
		return true;
	}

	@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

}

activity_main.xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:background="#E1E6F6"
    android:orientation="vertical" >

    <com.ac.flowlayout_danxuan.FlowLayout
	    	android:id="@+id/tag_layout"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:background="#ff00f0"
	        android:layout_marginTop="20dp">
    </com.ac.flowlayout_danxuan.FlowLayout>
	       
    <Button 
        android:id="@+id/btn_sure"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="添加"
        />
	
</LinearLayout>


项目源码：https://github.com/zqHero/FlowLayoutDemo





csdn：http://blog.csdn.net/u013233097/article/details/50724332
