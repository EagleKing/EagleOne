---
layout: post
title: 解决手势冲突
description: 解决项目中遇到的问题
image: assets/images/pic03.jpg
---

### 手势冲突
tableviewcell可以触发点击，同时tableview的父视图有点击识别，这样点击的时候就会产生冲突。解决方法在GestureRecgnizer代理方法里面区分手势。

***
~~~

    #pragma mark tapGestureRecgnizerdelegate 解决手势冲突
    - (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch {
    if ([touch.view isKindOfClass:[UITableView class]]){
        return NO;
    }
    if ([NSStringFromClass([touch.view class]) isEqualToString:@"UITableViewCellContentView"]) {
        return NO;
    }
    return YES;
    }
    
~~~

### 两个控件之间的手势冲突
我在一个横向滚动的scrollview里面加了一个竖向滚动的tableview，这时如果实现了scrollview的代理方法却没有区分scrollview和tableview，这时候tableview的滚动会发生混乱。解决方法是在didScroll代理方法里区分这两个。

***

~~~

    #pragma mark - scrollView delegate
    -(void)scrollViewDidScroll:(UIScrollView *)scrollView{
    if ([scrollView isKindOfClass:[UITableView class]]) {
       // NSLog(@"------是列表---");
     }
     else {
       // NSLog(@"------是滚动试图----");
    
     }
    }
    
~~~

2016/9/22更新

###  UIScrollView和子视图TableView的cell右滑删除冲突

横向滚动的scrollview里面有一个子视图tableview，tableview的cell右右滑冲突，除非手指激活tracking停留一会儿，否则无法激活右滑删除。
解决办法类似上面的，scrollview的左右滑依旧是由UIPanGesturerRecognizer控制的，但是该手势的代理无法更改

***   

~~~

     // Use these accessors to configure the scroll view's built-in gesture recognizers.
     // Do not change the gestures' delegates or override the getters for these properties.
     @property(nonatomic, readonly) UIPanGestureRecognizer *panGestureRecognizer NS_AVAILABLE_IOS(5_0);
     
~~~

***

写一个UIScrollView的子类重写下面的方法即可:

***

~~~

    -(BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch
    {
    //NSLog(@"手势触发的类＝%@",NSStringFromClass([touch.view class]));
    // 若为UITableViewCellContentView（即点击了tableViewCell），则不截获Touch事件
    if ([NSStringFromClass([touch.view class])isEqualToString:@"UITableViewCellContentView"]) {
        return NO;
    }
    return YES;
    }
    -(BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer
    {
    // 若为UITableViewCellContentView（即点击了tableViewCell），则不截获Touch事件
    if ([NSStringFromClass([gestureRecognizer.view class])isEqualToString:@"UITableViewCellContentView"]) {
        return NO;
    }
    return YES;
    }
    
~~~

