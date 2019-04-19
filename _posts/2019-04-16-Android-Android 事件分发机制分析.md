# Android 事件分发机制分析

## 前言

-  `Android`事件分发机制是`Android`开发者必须了解的基础
-  很多时候粗略了解总会有些遗漏，总是需要从头到尾梳理一遍，理清思路

> 1. 先通过基础认知，对分发机制有个初略了解，解开其神秘的面纱。然后再逐步深入，源码分析了解流程
> 2. 总结事件分发的流程图谱，对事件先后顺序有个清晰的认识
> 3. 结合现实开发中的一些案例，利用分发机制的知识定制化实现目标



![示意图](https://ws3.sinaimg.cn/large/006tNc79gy1g1zo74pqfej317k0nyjte.jpg)

## 1. 基础认知

### 1.1 事件分发的对象是谁？

答：点击事件（`Touch`事件）

- 定义 
  当用户触摸屏幕时（`View` 或 `ViewGroup`派生的控件），将产生点击事件（`Touch`事件）

  > `Touch`事件的相关细节（发生触摸的位置、时间等）被封装成`MotionEvent`对象

- 事件类型（4种）

| 事件类型                  | 具体动作                   |
| ------------------------- | -------------------------- |
| MotionEvent.ACTION_DOWN   | 按下View（所有事件的开始） |
| MotionEvent.ACTION_UP     | 抬起View（与DOWN对应）     |
| MotionEvent.ACTION_MOVE   | 滑动View                   |
| MotionEvent.ACTION_CANCEL | 结束事件（非人为原因）     |

- 特别说明：事件列

从手指接触屏幕 至 手指离开屏幕，这个过程产生的一系列事件

>  注：一般情况下，事件列都是以`DOWN`事件开始、`UP`事件结束，中间有无数的`MOVE`事件，如下图： 

![](https://ws2.sinaimg.cn/large/006tNc79ly1g24hgebzfjj30k107zjrj.jpg)


即当一个点击事件（`MotionEvent` ）产生后，系统需把这个事件传递给一个具体的 View 去处理。

### 1.2 事件分发的本质

答：将点击事件（`MotionEvent`）传递到某个具体的View & 处理的整个过程

> 即 事件传递的过程 = 分发过程。

### 1.3 事件在哪些对象之间进行传递？

答：`Activity`、`ViewGroup`、`View`

`Android`的`UI`界面由`Activity`、`ViewGroup`、`View` 及其派生类组成 

![时间传递设计的组件](https://ws2.sinaimg.cn/large/006tNc79ly1g24s1xkchij305t0ad3yg.jpg)

# 2.流程分析

从 activity 开始触摸屏幕，分发到ViewGroup View的流程



![事件分发流程图](https://ws1.sinaimg.cn/large/006tNc79ly1g24t4egzaqj30ns0o0gnw.jpg)

---------------------

通过增加打印日志，分别在下列方法开始执行的时候打印出相关文字，对流程进行简单的分析判断

-·Activity

    > 1. dispatchTouchEvent
    > 2. onTouchEvent

- ViewGroup 
   > 1. dispatchTouchEvent
   > 2. onFilterTouchEventForSecurity
   > 3. onInterceptTouchEvent
   > 4. onTouchEvent

- View
   > 1. dispatchTouchEvent
   > 2. onFilterTouchEventForSecurity
   > 3. onTouchEvent

#### 1.1 默认都不拦截的请求流程

```
I/activity: down----------dispatchTouchEvent
I/viewGroup: down----------dispatchTouchEvent
I/viewGroup: down----------onFilterTouchEventForSecurity
I/viewGroup: down----------onInterceptTouchEvent
I/View: down----------dispatchTouchEvent
I/View: down----------onFilterTouchEventForSecurity
I/View: down----------onTouchEvent
I/viewGroup: down----------onFilterTouchEventForSecurity
I/viewGroup: down----------onTouchEvent
I/activity: down----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/activity: move----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/activity: move----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/activity: move----------onTouchEvent
I/activity: up----------dispatchTouchEvent
I/activity: up----------onTouchEvent
```



#### 1.2 ViewGroup Intercept 拦截事件 的请求流程

```
I/activity: down----------dispatchTouchEvent
I/viewGroup: down----------dispatchTouchEvent
I/viewGroup: down----------onFilterTouchEventForSecurity
I/viewGroup: down----------onInterceptTouchEvent
I/viewGroup: down----------onFilterTouchEventForSecurity
I/viewGroup: down----------onTouchEvent
I/activity: down----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/activity: move----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/activity: move----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/activity: move----------onTouchEvent
I/activity: up----------dispatchTouchEvent
I/activity: up----------onTouchEvent
```



#### 1.3 View 消费事件

```
I/activity: down----------dispatchTouchEvent
I/viewGroup: down----------dispatchTouchEvent
I/viewGroup: down----------onFilterTouchEventForSecurity
I/viewGroup: down----------onInterceptTouchEvent
I/View: down----------dispatchTouchEvent
I/View: down----------onFilterTouchEventForSecurity
I/View: down----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/viewGroup: move----------dispatchTouchEvent
I/viewGroup: move----------onFilterTouchEventForSecurity
I/viewGroup: move----------onInterceptTouchEvent
I/View: move----------dispatchTouchEvent
I/View: move----------onFilterTouchEventForSecurity
I/View: move----------onTouchEvent
I/activity: move----------dispatchTouchEvent
I/viewGroup: move----------dispatchTouchEvent
I/viewGroup: move----------onFilterTouchEventForSecurity
I/viewGroup: move----------onInterceptTouchEvent
I/View: move----------dispatchTouchEvent
I/View: move----------onFilterTouchEventForSecurity
I/View: move----------onTouchEvent
I/activity: up----------dispatchTouchEvent
I/viewGroup: up----------dispatchTouchEvent
I/viewGroup: up----------onFilterTouchEventForSecurity
I/viewGroup: up----------onInterceptTouchEvent
I/View: up----------dispatchTouchEvent
I/View: up----------onFilterTouchEventForSecurity
I/View: up----------onTouchEvent
```

---------------------

# 3.源码分析

### 1. 从 Activity 到 ViewGroup 的dispatchTouchEvent

从`Activity` 的 `dispatchTouchEvent` 开始分发

```java
/**
	activity 的 dispatchTouchEvent方法，调用 getWindow().superDispatchTouchEvent(ev) 方法
 */
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}

```

调用phoneWindow 的 superDispatchTouchEvent 继续分发到 phoneWindow ,如果该方法放回true则不调用 onTouchEvent否则分发到自己的onTouchEvent方法

**PhoneWindow**

```java
///phoneWindow 的superDispatchTouchEvent中调用了mDecorView 的superDispatchTouchEvent
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```
**DecorView:ViewGroup**

```java
///调用父类 viewgroup 的 dispatchTouchEvent
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```

#### 2. ViewGroup 的 dispatchTouchEvent 进行事件分发

**ViewGroup**

```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
    }

    // If the event targets the accessibility focused view and this is it, start
    // normal event dispatch. Maybe a descendant is what will handle the click.
    if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
        ev.setTargetAccessibilityFocus(false);
    }

    boolean handled = false;
  	/// 判断当前事件是否应该被过滤掉，true 说明应该被分发
    if (onFilterTouchEventForSecurity(ev)) {
        final int action = ev.getAction();
      	///通过掩码获取基础事件信息,掩码ACTION_MASK=0xff 。比如 ACTION_POINTER_2_DOWN =0x0100 与 0xff 结果为 0x00 =
        final int actionMasked = action & MotionEvent.ACTION_MASK;

     		 /*******省略代码一：当事件为down事件时候，发送取消action 取消之前的事件。重置触摸状态*******/
       
        // Check for interception.
        final boolean intercepted;
      	///是down事件 或者有手指在触摸状态了 意思是
        if (actionMasked == MotionEvent.ACTION_DOWN
                || mFirstTouchTarget != null) {
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            if (!disallowIntercept) {
              	/// 执行onInterceptTouchEvent代码，判断是否拦截事件，给intercepted 赋值
                intercepted = onInterceptTouchEvent(ev);
                ev.setAction(action); // restore action in case it was changed
            } else {
                intercepted = false;
            }
        } else {
            // There are no touch targets and this action is not an initial down
            // so this view group continues to intercept touches.
            intercepted = true;
        }

        // If intercepted, start normal event dispatch. Also if there is already
        // a view that is handling the gesture, do normal event dispatch.
        if (intercepted || mFirstTouchTarget != null) {
            ev.setTargetAccessibilityFocus(false);
        }

        // Check for cancelation.
        final boolean canceled = resetCancelNextUpFlag(this)
                || actionMasked == MotionEvent.ACTION_CANCEL;

        // Update list of touch targets for pointer down, if needed.
        final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
        TouchTarget newTouchTarget = null;
        boolean alreadyDispatchedToNewTouchTarget = false;
        if (!canceled && !intercepted) {

            // If the event is targeting accessibility focus we give it to the
            // view that has accessibility focus and if it does not handle it
            // we clear the flag and dispatch the event to all children as usual.
            // We are looking up the accessibility focused host to avoid keeping
            // state since these events are very rare.
            View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                    ? findChildWithAccessibilityFocus() : null;

            if (actionMasked == MotionEvent.ACTION_DOWN
                    || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                final int actionIndex = ev.getActionIndex(); // always 0 for down
                final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                        : TouchTarget.ALL_POINTER_IDS;

                // Clean up earlier touch targets for this pointer id in case they
                // have become out of sync.
                removePointersFromTouchTargets(idBitsToAssign);

                final int childrenCount = mChildrenCount;
                if (newTouchTarget == null && childrenCount != 0) {
                    final float x = ev.getX(actionIndex);
                    final float y = ev.getY(actionIndex);
                    // Find a child that can receive the event.
                    // Scan children from front to back.
                    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                    final boolean customOrder = preorderedList == null
                            && isChildrenDrawingOrderEnabled();
                    final View[] children = mChildren;
                    for (int i = childrenCount - 1; i >= 0; i--) {
                        final int childIndex = getAndVerifyPreorderedIndex(
                                childrenCount, i, customOrder);
                        final View child = getAndVerifyPreorderedView(
                                preorderedList, children, childIndex);

                        // If there is a view that has accessibility focus we want it
                        // to get the event first and if not handled we will perform a
                        // normal dispatch. We may do a double iteration but this is
                        // safer given the timeframe.
                        if (childWithAccessibilityFocus != null) {
                            if (childWithAccessibilityFocus != child) {
                                continue;
                            }
                            childWithAccessibilityFocus = null;
                            i = childrenCount - 1;
                        }

                        if (!canViewReceivePointerEvents(child)
                                || !isTransformedTouchPointInView(x, y, child, null)) {
                            ev.setTargetAccessibilityFocus(false);
                            continue;
                        }

                        newTouchTarget = getTouchTarget(child);
                        if (newTouchTarget != null) {
                            // Child is already receiving touch within its bounds.
                            // Give it the new pointer in addition to the ones it is handling.
                            newTouchTarget.pointerIdBits |= idBitsToAssign;
                            break;
                        }

                        resetCancelNextUpFlag(child);
                        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                            // Child wants to receive touch within its bounds.
                            mLastTouchDownTime = ev.getDownTime();
                            if (preorderedList != null) {
                                // childIndex points into presorted list, find original index
                                for (int j = 0; j < childrenCount; j++) {
                                    if (children[childIndex] == mChildren[j]) {
                                        mLastTouchDownIndex = j;
                                        break;
                                    }
                                }
                            } else {
                                mLastTouchDownIndex = childIndex;
                            }
                            mLastTouchDownX = ev.getX();
                            mLastTouchDownY = ev.getY();
                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
                            alreadyDispatchedToNewTouchTarget = true;
                            break;
                        }

                        // The accessibility focus didn't handle the event, so clear
                        // the flag and do a normal dispatch to all children.
                        ev.setTargetAccessibilityFocus(false);
                    }
                    if (preorderedList != null) preorderedList.clear();
                }

                if (newTouchTarget == null && mFirstTouchTarget != null) {
                    // Did not find a child to receive the event.
                    // Assign the pointer to the least recently added target.
                    newTouchTarget = mFirstTouchTarget;
                    while (newTouchTarget.next != null) {
                        newTouchTarget = newTouchTarget.next;
                    }
                    newTouchTarget.pointerIdBits |= idBitsToAssign;
                }
            }
        }

        // Dispatch to touch targets.
        if (mFirstTouchTarget == null) {
            // No touch targets so treat this as an ordinary view.
            handled = dispatchTransformedTouchEvent(ev, canceled, null,
                    TouchTarget.ALL_POINTER_IDS);
        } else {
            // Dispatch to touch targets, excluding the new touch target if we already
            // dispatched to it.  Cancel touch targets if necessary.
            TouchTarget predecessor = null;
            TouchTarget target = mFirstTouchTarget;
            while (target != null) {
                final TouchTarget next = target.next;
                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                    handled = true;
                } else {
                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
                            || intercepted;
                    if (dispatchTransformedTouchEvent(ev, cancelChild,
                            target.child, target.pointerIdBits)) {
                        handled = true;
                    }
                    if (cancelChild) {
                        if (predecessor == null) {
                            mFirstTouchTarget = next;
                        } else {
                            predecessor.next = next;
                        }
                        target.recycle();
                        target = next;
                        continue;
                    }
                }
                predecessor = target;
                target = next;
            }
        }

        // Update list of touch targets for pointer up or cancel, if needed.
        if (canceled
                || actionMasked == MotionEvent.ACTION_UP
                || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
            resetTouchState();
        } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
            final int actionIndex = ev.getActionIndex();
            final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
            removePointersFromTouchTargets(idBitsToRemove);
        }
    }

    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
    }
    return handled;
}
```