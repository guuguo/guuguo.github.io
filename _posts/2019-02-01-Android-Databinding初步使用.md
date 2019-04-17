---
layout:     post
title:      databinding 的大概使用
date:       2019-02-01
author:     guuguo
catalog: true
tags:
    - Android
    - MVVM

---

# databinding 的大概使用

#### 1.准备

- module 的 build.gradle 的 android 节点下面启用 dataBinding

```groovy
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

- apply `kapt` 插件 `apply plugin: 'kotlin-kapt'`
- `dependencies` 内添加依赖

```groovy
kapt "com.android.databinding:compiler:${ANDROID_PLUGIN_VERSION}"
```

`ANDROID_PLUGIN_VERSION` 为 `com.android.tools.build:gradle`同样的版本号

> 接下来同步一下就可以使用了

#### 2.基础用法

- xml 改动

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

       <!--实体-->
       <variable
            name="bean"
            type="com.bianla.dataserviceslibrary.domain.ModulesBean" />

       <!--类引用--> <import type="android.view.View" />
    </data>
    <!--布局文件...-->
</layout
```

- 布局使用

```java
android:visibility="@{bean.getFatReportVisible()}" //bean中简单处理一些相关逻辑

android:visibility="@{bean.isVisible?View.VISIBLE:View.GONE}" //三目运算符判断

android:onClick="@{viewModel::onMoreRecommendStarClick}" //点击事件

android:text="@{viewModel.hotSearchStr}" //字符串表示

android:text="@{bean.clickCount+@string/community_play_times}" //10次播放(中文直接+再后面 windows 编译报错, linux 和 mac 正常。需要使用 @string 资源方式)
```



> 自定义属性(随便找个类写静态方法，标记注解)

```kotlin

//kotlin写法
@BindingAdapter("home_img_url")
fun ImageView.setImage(imageUrl: String?) {
    Glide.with(this.context).load(imageUrl)
            .apply(RequestOptions().placeholder(R.drawable.community_default_image).dontAnimate())
            .into(this)
}

//java写法
@BindingAdapter("home_img_url")
public static void setImage(view ImageView,imageUrl: String?) {
    Glide.with(view.context).load(imageUrl)
            .apply(RequestOptions().placeholder(R.drawable.community_default_image).dontAnimate())
            .into(view)
}

```

> 自定义属性多参数

```kotlin
@BindingAdapter("avatarUrl", "gender")
fun ImageView.setAvatar(url: String?, gender: Boolean) {
    Glide.with(this.context).load(url)
            .apply(avatarGlideOptions(gender))
            .into(this)
}

```

> 自定义属性与自定义 view 的自有方法绑定方式

```kotlin
@BindingMethods(BindingMethod(type = FunctionTextView::class, attribute = "android:text", method = "setText"),
        BindingMethod(type = FunctionTextView::class, attribute = "android:textColor", method = "setTextColor"),
        BindingMethod(type = FunctionTextView::class, attribute = "ftv_drawableSrc", method = "setDrawable"),
        BindingMethod(type = FunctionTextView::class, attribute = "ftv_drawableTint", method = "setDrawableTint"),
        BindingMethod(type = RoundedImageView::class, attribute = "riv_border_width", method = "setBorderWidth")
        )
object ViewBindingAdapter {
}
```

#### 3.MVVM

> **View: **对应于 Activity 和 xml ，负责 View 的绘制以及与用户交互
>
> **Model: **实体模型(可包含一些简单的逻辑)
>
> **ViewModel: **负责完成 View 于 Model 间的交互,负责业务逻辑

###### 我的项目架构

> IView

```kotlin
interface IView<E> : LifecycleProvider<E> {
    fun loadData()

    fun dialogLoadingShow(msg: String, canTouchCancel: Boolean = false, maxDelay: Long = 0, listener: DialogInterface.OnDismissListener? = null)

    fun dialogErrorShow(msg: String, listener: DialogInterface.OnDismissListener? = null, delayTime: Int = 1500)

    fun dialogCompleteShow(msg: String, listener: DialogInterface.OnDismissListener? = null, delayTime: Int = 800)

    fun dialogMsgShow(msg: String, btnText: String, listener: (() -> Unit)?): WarningDialog?

    fun dialogWarningShow(msg: String, cancelStr: String, confirmStr: String, listener: (() -> Unit)? = null): WarningDialog?

    fun showDialogOnMain(dialog: Dialog)

    fun dialogDismiss()

    var activity: LBaseActivitySupport
}
```

> BaseViewModel

```kotlin
open class BaseViewModel(view: IView<*>?=null) : BaseObservable() {

    var showLoading: (msg: String) -> Unit = { msg -> view?.dialogLoadingShow(msg) }
    var hideLoading: () -> Unit = { view?.dialogDismiss() }
    var showErrorMsg: (msg: String) -> Unit = { msg -> view?.dialogErrorShow(msg) }
    var showComplete: (msg: String) -> Unit = { msg -> view?.dialogCompleteShow(msg) }
    fun getVisibility(bool: Boolean): Int {
        return if (bool)
            View.VISIBLE
        else View.GONE
    }
}
```

> ViewModel

```kotlin
class CommunityDataViewModel(var view: IView<*>) : BaseViewModel(view) {
    val activity = view.activity
    var setUpRecommendList: (list: ResRecommendList) -> Unit = { }
    var setUpThemeData: (list: HomeThemesBean) -> Unit = { }
    var setUpCommunityDataList: (data: Pair<ResRecommendList, Int>) -> Unit = {}

    var loadMoreFaild: () -> Unit = { }

    fun getRecommendList(page: Int, noCache: Boolean = false) =
            CApiServer.getRecommendList(page)
                    .compose(NetTransform())
                    .compose(ACacheTransform<BaseEntity<ResRecommendList>>("RecommendList$page").also { if (noCache) it.noCache() else it.fromCacheAndNet() })
}
```

> java 方式观察者

```java
mainViewModel.isShowDialog.addOnPropertyChangedCallback(new android.databinding.Observable.OnPropertyChangedCallback() {
      @Override
      public void onPropertyChanged(android.databinding.Observable sender, int propertyId) {
          if (mainViewModel.isShowDialog.get()) {
               dialog.show();
          } else {
               dialog.dismiss();
          }
       }
    });
 }
```
