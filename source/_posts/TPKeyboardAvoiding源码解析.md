---
title: TPKeyboardAvoiding源码解析
date: 2019-8-30 16:35:40
tag: iOS
categories: 源码阅读
---

#### 简介 : TPKeyboardAvoiding是一个第三方键盘管理工具，作用是避免键盘遮挡视图。

#### 框架结构 ![TPKeyboardAvoiding](https://tva1.sinaimg.cn/large/006y8mN6gy1g6kxpmgzg6j30jp082aal.jpg)

其中一共有3个类,1个分类。

#### 框架使用

![image-20190830170555434](https://tva1.sinaimg.cn/large/006y8mN6gy1g6kxq56c6tj31eo08edh7.jpg)

只需要将用到tableView的地方替换成TPKeyboardAvoidingTableView，同理适用于TPKeyboardAvoidingCollectionView，TPKeyboardAvoidingScrollView。其中整个核心都是利用UIScrollView (TPKeyboardAvoidingAdditions)来实现的。

#### 源码解析

由于整个核心实现都是在以下代码中实现的，所以主要针对以下两个方法进行讲解。

```objc
- (void)TPKeyboardAvoiding_keyboardWillShow:(NSNotification*)notification;
- (void)TPKeyboardAvoiding_keyboardWillHide:(NSNotification*)notification;
```



```objc
- (void)TPKeyboardAvoiding_keyboardWillShow:(NSNotification*)notification {
    NSDictionary *info = [notification userInfo];
    TPKeyboardAvoidingState *state = self.keyboardAvoidingState;
    
    state.animationDuration = [[info objectForKey:kUIKeyboardAnimationDurationUserInfoKey] doubleValue];

    CGRect keyboardRect = [self convertRect:[[info objectForKey:_UIKeyboardFrameEndUserInfoKey] CGRectValue] fromView:nil];
    if (CGRectIsEmpty(keyboardRect)) {
        return;
    }
    
    if ( state.ignoringNotifications ) {
        return;
    }

    state.keyboardRect = keyboardRect;

    if ( !state.keyboardVisible ) {
        state.priorInset = self.contentInset;
        state.priorScrollIndicatorInsets = self.scrollIndicatorInsets;
        state.priorPagingEnabled = self.pagingEnabled;
    }

    state.keyboardVisible = YES;
    self.pagingEnabled = NO;

    if ( [self isKindOfClass:[TPKeyboardAvoidingScrollView class]] ) {
        state.priorContentSize = self.contentSize;
        self.contentSize = CGSizeZero;
        
        if ( CGSizeEqualToSize(self.contentSize, CGSizeZero) ) {
            // Set the content size, if it's not set. Do not set content size explicitly if auto-layout
            // is being used to manage subviews
            self.contentSize = [self TPKeyboardAvoiding_calculatedContentSizeFromSubviewFrames];
        }
    }
    
    // Delay until a future run loop such that the cursor position is available in a text view
    // In other words, it's not available (specifically, the prior cursor position is returned) when the first keyboard position change notification fires
    // NOTE: Unfortunately, using dispatch_async(main_queue) did not result in a sufficient-enough delay
    // for the text view's current cursor position to be available
    dispatch_time_t delay = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.01 * NSEC_PER_SEC));
    
    dispatch_after(delay, dispatch_get_main_queue(), ^{
        
        // Shrink view's inset by the keyboard's height, and scroll to show the text field/view being edited
        [UIView beginAnimations:nil context:NULL];
        
        [UIView setAnimationDelegate:self];
        [UIView setAnimationWillStartSelector:@selector(keyboardViewAppear:context:)];
        
        [UIView setAnimationDidStopSelector:@selector(keyboardViewDisappear:finished:context:)];
        
        [UIView setAnimationCurve:[[[notification userInfo] objectForKey:UIKeyboardAnimationCurveUserInfoKey] intValue]];
        [UIView setAnimationDuration:[[[notification userInfo] objectForKey:UIKeyboardAnimationDurationUserInfoKey] floatValue]];
        
        self.contentInset = [self TPKeyboardAvoiding_contentInsetForKeyboard];
        
        UIView *firstResponder = [self TPKeyboardAvoiding_findFirstResponderBeneathView:self];
        if ( firstResponder ) {
            CGFloat viewableHeight = self.bounds.size.height - self.contentInset.top - self.contentInset.bottom;
            [self setContentOffset:CGPointMake(self.contentOffset.x,
                                               [self TPKeyboardAvoiding_idealOffsetForView:firstResponder
                                                                     withViewingAreaHeight:viewableHeight])
                          animated:NO];
        }
        
        self.scrollIndicatorInsets = self.contentInset;
        [self layoutIfNeeded];
        
        [UIView commitAnimations];
    });
}
```

该方法主要分为以下几个步骤:

1、拿到键盘将要出现的状态；

2、保存当前ScrollView的状态；

3、计算当前ScrollView滚动范围；

4、调整ScrollView的contentInset；

5、调整ScrollView的ContentOffset。

```objc
- (void)TPKeyboardAvoiding_keyboardWillHide:(NSNotification*)notification {
    CGRect keyboardRect = [self convertRect:[[[notification userInfo] objectForKey:_UIKeyboardFrameEndUserInfoKey] CGRectValue] fromView:nil];
    if (CGRectIsEmpty(keyboardRect) && !self.keyboardAvoidingState.keyboardAnimationInProgress) {
        return;
    }
    
    TPKeyboardAvoidingState *state = self.keyboardAvoidingState;
    
    if ( state.ignoringNotifications ) {
        return;
    }
    
    if ( !state.keyboardVisible ) {
        return;
    }
    
    state.keyboardRect = CGRectZero;
    state.keyboardVisible = NO;
    
    // Restore dimensions to prior size
    [UIView beginAnimations:nil context:NULL];
    [UIView setAnimationCurve:[[[notification userInfo] objectForKey:UIKeyboardAnimationCurveUserInfoKey] intValue]];
    [UIView setAnimationDuration:[[[notification userInfo] objectForKey:UIKeyboardAnimationDurationUserInfoKey] floatValue]];
    
    if ( [self isKindOfClass:[TPKeyboardAvoidingScrollView class]] ) {
        self.contentSize = state.priorContentSize;
    }
    
    self.contentInset = state.priorInset;
    self.scrollIndicatorInsets = state.priorScrollIndicatorInsets;
    self.pagingEnabled = state.priorPagingEnabled;
	[self layoutIfNeeded];
    [UIView commitAnimations];
}
```

该方法主要负责ScrollView状态的恢复。

至此，该框架的核心方法已经总结完毕。

#### 小结

TPKeyboardAvoiding主要调整contentInset和contentOffset来解决键盘的遮挡问题。以后如果需要自己设计一个解决键盘遮挡问题的框架，便可以参考此思路。
