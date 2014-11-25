---
layout: post
title: "去除布局时的隐式动画效果"
date: 2014-11-25 16:51:18 +0800
comments: true
categories: 细节必究
---
原创，欢迎转载，转载请注明出处。

在布局UITableViewCell子试图时，如果在layoutSubviews中处理布局，当项目运行时我们经常会看到cell上面的控件在动，这也就是标题所说的隐式动画。虽然说动态效果更能够吸引人，但有时候并非我们所要，那我们应该怎样解决这个问题呢？

<!--more-->

解决方法代码示例如下：

**swift版：**

```
override func layoutSubviews() {
        super.layoutSubviews()
        CATransaction.begin()
        CATransaction.setValue(kCFBooleanTrue, forKey: kCATransactionDisableActions)
        /*******************/
        // 在这里书写布局的代码
        /*******************/
        CATransaction.commit()
    }
```
    
**OC版：**

```
- (void)layoutSubviews
{
    [super layoutSubviews];    
    [CATransaction begin];
    [CATransaction setValue:(id)kCFBooleanTrue forKey:kCATransactionDisableActions];
    /*******************/
    // 在这里书写布局的代码
    /*******************/
    [CATransaction commit];
}
```
**注意：**OC版kCFBooleanTrue前面需要添加id,否则汇报警告。

从上面的方法可以看出，只需要在布局的代码前面加上`CATransaction.begin()`和`CATransaction.setValue(kCFBooleanTrue, forKey: kCATransactionDisableActions)`，后面加上`CATransaction.commit()`就可以解决这个问题了。