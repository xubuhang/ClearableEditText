# ClearableEditText
Example of ClearableEditText

在Android的输入框中加入清除按钮，是很常见的设计，本文介绍如何创建一个控件，在输入框中加入清除按钮。


我们来看看实现这个控件都需要做什么：

清除按钮在输入框中有内容时出现
清除按钮必须出现在输入框内
点击清除按钮，清除输入框中的所有内容
清除按钮的颜色必须与主题一致
实现第一点，我们可以通过加入TextWatcher来监听EditText的变化，在onFocusChangeListener方法中处理清除按钮是否可见。
实现第二点，我们需要使用compound drawable作为清除按钮，然后在 OnTouch listener中处理点击事件。

开始实现我们的EditText
我们使用AppCompatEditText作为基类


public class ClearableEditText extends AppCompatEditText
implements View.OnTouchListener, View.OnFocusChangeListener, TextWatcher {

接着加入构造函数

public ClearableEditText(final Context context) {
    super(context);
    init(context);
}
 
public ClearableEditText(final Context context, final AttributeSet attrs) {
    super(context, attrs);
    init(context);
}
 
public ClearableEditText(final Context context, final AttributeSet attrs, final int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    init(context);
}

实现init方法

创建drawable，并为其加入Touch、Focus事件处理
加入TextChangedListener，监听EditText内容变化

private void init(final Context context) {
    final Drawable drawable = ContextCompat.getDrawable(context, R.drawable.abc_ic_clear_mtrl_alpha);
    final Drawable wrappedDrawable = DrawableCompat.wrap(drawable); //Wrap the drawable so that it can be tinted pre Lollipop
    DrawableCompat.setTint(wrappedDrawable, getCurrentHintTextColor());
    mClearTextIcon = wrappedDrawable;
    mClearTextIcon.setBounds(0, 0, mClearTextIcon.getIntrinsicHeight(), mClearTextIcon.getIntrinsicHeight());
    setClearIconVisible(false);
    super.setOnTouchListener(this);
    super.setOnFocusChangeListener(this);
    addTextChangedListener(this);
}

我们默认使用setClearIconVisible(false)隐藏了清除按钮，在输入文本时才会显示


private void setClearIconVisible(final boolean visible) {
    mClearTextIcon.setVisible(visible, false);
    final Drawable[] compoundDrawables = getCompoundDrawables();
    setCompoundDrawables(
            compoundDrawables[0],
            compoundDrawables[1],
            visible ? mClearTextIcon : null,
            compoundDrawables[3]);
}

回到顶部
加入Listener

private Drawable mClearTextIcon;
private OnFocusChangeListener mOnFocusChangeListener;
private OnTouchListener mOnTouchListener;
 
@Override
public void setOnFocusChangeListener(final OnFocusChangeListener onFocusChangeListener) {
    mOnFocusChangeListener = onFocusChangeListener;
}
 
@Override
public void setOnTouchListener(final OnTouchListener onTouchListener) {
    mOnTouchListener = onTouchListener;
}
回到顶部
实现Listener
最后我们来实现3个Listener，先来看focus Listener


@Override
public void onFocusChange(final View view, final boolean hasFocus) {
    if (hasFocus) {
        setClearIconVisible(getText().length() > 0);
    } else {
        setClearIconVisible(false);
    }
    if (mOnFocusChangeListener != null) {
        mOnFocusChangeListener.onFocusChange(view, hasFocus);
    }
}

在获取焦点时，判断输入框中内容是否大于0，有内容则显示清除按钮。

接着我们来看onTouch方法：


@Override
public boolean onTouch(final View view, final MotionEvent motionEvent) {
    final int x = (int) motionEvent.getX();
    if (mClearTextIcon.isVisible() && x > getWidth() - getPaddingRight() - mClearTextIcon.getIntrinsicWidth()) {
        if (motionEvent.getAction() == MotionEvent.ACTION_UP) {
            setText("");
        }
        return true;
    }
    return mOnTouchListener != null && mOnTouchListener.onTouch(view, motionEvent);
}

在这里，我们首先检查了清除按钮是否为显示状态，然后判断点击的范围是否在清除按钮内，
如果在范围内的话，在ACTION_UP时清空输入框内容，否则执行mOnTouchListener的
onTouch方法。

最后，我们实现TextWatcher：


@Override
public final void onTextChanged(final CharSequence s, final int start, final int before, final int count) {
    if (isFocused()) {
        setClearIconVisible(s.length() > 0);
    }
}
 
@Override
public void beforeTextChanged(CharSequence s, int start, int count, int after) {
}
 
@Override
public void afterTextChanged(Editable s) {
}

判断输入框中的字数，大于0则显示清除按钮，否则隐藏。

如果你使用的是AutoCompleteTextView，我们也可以使用同样的方法添加清除按钮：


public class ClearableAutoCompleteTextView extends AppCompatAutoCompleteTextView
implements View.OnTouchListener, View.OnFocusChangeListener, TextWatcher {

该控件的源码已上传到Github: ClearableEditText
