---
layout: post
title: "iOS8下的UIActionSheet"
date: 2014-11-28 20:43:02 +0800
comments: false
categories: iOS8、问题集
---
<br><br/>
前段时间做了一个iPad应用，里面用到了图片上传。因此，需要用到UIImagePickerController打开本地图库和开启相机拍照。代码如下：

``` ruby
- (IBAction)choosePhoto:(id)sender
{
    UIActionSheet *sheet = [[UIActionSheet alloc] initWithTitle:@"选择照片"
                                                       delegate:self
                                              cancelButtonTitle:@"取消"
                                         destructiveButtonTitle:@"照片库"
                                              otherButtonTitles:@"拍一张", nil];
    [sheet showInView:self.view];
}
- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex
{
    if (buttonIndex == actionSheet.cancelButtonIndex) {
        return;
    }
    UIImagePickerController *picker = [[UIImagePickerController alloc]init];
    if (buttonIndex == 1) {
        [picker setSourceType:UIImagePickerControllerSourceTypePhotoLibrary];
    }else{
        [picker setSourceType:UIImagePickerControllerSourceTypeCamera];
    }
    [self presentViewController:picker animated:YES completion:nil];
}
```

<br><br/>

在iOS 7之前的模拟器以及iOS 7模拟器上测试都没问题。但是，iOS 8上报了如下警告：


>   Warning: Attempt to present <UIImagePickerController: 0x7f96c28f7800>  on <ViewController: 0x7f96c302c3d0> which is already presenting <UIAlertController: 0x7f96c337be90>

<br><br/>
根据问题可以看出ViewController在present UIImagePickerController之前present了UIAlertController，但是，我并没有是用UIAlertView。但是这里出现了UIAlertController这个iOS 8推出的新的类，问题应该就出现在这里。然后，我在UIActionSheet头文件找到了这样一句话：

>   UIActionSheet is deprecated. Use UIAlertController
>    with a preferredStyle of UIAlertControllerStyleActionSheet instead  

<br><br/>
原来，在iOS 8下UIActionSheet已经被废弃掉，取而代之的是UIAlertController。

于是，我将代码修改如下：

```
// 头部添加全局actionSheet
@property (strong ,nonatomic) UIAlertController *actionSheet;
// 显示actionSheet
- (IBAction)choosePhoto:(id)sender
{
    [self presentViewController:self.actionSheet 
                       animated:YES
                     completion:nil];
}
- (UIAlertController *)actionSheet
{
    if (_actionSheet == nil) {
        _actionSheet = [UIAlertController alertControllerWithTitle:@"选择照片"  
                                                           message:@"请通过以下方式来选择照片" 
                                                                                         preferredStyle:UIAlertControllerStyleActionSheet];
        // 在action sheet中，UIAlertActionStyleCancel不起作用
        UIAlertAction *act1 = [UIAlertAction actionWithTitle:@"相机" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
        }];
        UIAlertAction *act2 = [UIAlertAction actionWithTitle:@"图库" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
        }];
        UIAlertAction *act3 = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleDestructive handler:^(UIAlertAction *action) {
        }];
        [_actionSheet addAction:act1];
        [_actionSheet addAction:act2];
        [_actionSheet addAction:act3];
    }
    return _actionSheet;
}
```
<br><br/>
结果，运行时程序直接崩掉了。错误信息：

>   Terminating app due to uncaught exception 'NSGenericException', reason: 'Your application has presented a UIAlertController (<UIAlertController: 0x7f8e5841e790>) of style UIAlertControllerStyleActionSheet. The modalPresentationStyle of a UIAlertController with this style is UIModalPresentationPopover. You must provide location information for this popover through the alert controller's popoverPresentationController. You must provide either a sourceView and sourceRect or a barButtonItem.  If this information is not known when you present the alert controller, you may provide it in the UIPopoverPresentationControllerDelegate method -prepareForPopoverPresentation.'

由错误信息已经详细的描述了问题所在，以及解决方法。也就是当UIAlertController以UIAlertControllerStyleActionSheet类型展示的时候，必须借助于UIPopoverPresentationController这个类。


**注意：**[UIPopoverPresentationController](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIPopoverPresentationController_class/index.html)是[UIPresentationController](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIPresentationController_class/index.html)的一个子类，此类将取代UIPopoverController，并且可以再iPhone应用中实现以前只有iPad才有的UIPopoverController功能。

<br><br/>
于是，将上面的代码修改如下：

```
- (IBAction)choosePhoto:(id)sender
{
    UIPopoverPresentationController *pop = self.actionSheet.popoverPresentationController;
    pop.delegate = self;
    pop.sourceView = self.view;
    // 显示在中心位置
    pop.sourceRect = CGRectMake((CGRectGetWidth(pop.sourceView.bounds)-2)*0.5f, (CGRectGetHeight(pop.sourceView.bounds)-2)*0.5f, 2, 2);
    [self presentViewController:self.actionSheet animated:YES completion:nil];
}
#pragma mark - UIPopoverPresentationControllerDelegate
- (void)prepareForPopoverPresentation:(UIPopoverPresentationController *)popoverPresentationController
{
    NSLog(@"%s", __PRETTY_FUNCTION__);
}
// Called on the delegate when the popover controller will dismiss the popover. Return NO to prevent the
// dismissal of the view.
- (BOOL)popoverPresentationControllerShouldDismissPopover:(UIPopoverPresentationController *)popoverPresentationController
{
    NSLog(@"%s", __PRETTY_FUNCTION__);
    return YES;
}
// Called on the delegate when the user has taken action to dismiss the popover. This is not called when the popover is dimissed programatically.
- (void)popoverPresentationControllerDidDismissPopover:(UIPopoverPresentationController *)popoverPresentationController
{
    NSLog(@"%s", __PRETTY_FUNCTION__);
}
// -popoverPresentationController:willRepositionPopoverToRect:inView: is called on your delegate when the popover may require a different view or rectangle.
- (void)popoverPresentationController:(UIPopoverPresentationController *)popoverPresentationController
          willRepositionPopoverToRect:(inout CGRect *)rect
                               inView:(inout UIView **)view
{
    NSLog(@"%s", __PRETTY_FUNCTION__);
}
```

<br><br/>
此时，运行结果如下：<br><br/>

![](https://raw.githubusercontent.com/jixuqianxing/jixuqianxing.github.com/master/images/blogImages/20141128/20141128_1.png)

<br><br/>
但是，当我旋转屏幕时，问题又出现了。<br><br/>

![](https://raw.githubusercontent.com/jixuqianxing/jixuqianxing.github.com/master/images/blogImages/20141128/20141128_2.gif)

<br><br/>
从图中可以看出，actionSheet的位置不在使我们想要的。解决办法如下：

```
- (void)popoverPresentationController:(UIPopoverPresentationController *)popoverPresentationController
          willRepositionPopoverToRect:(inout CGRect *)rect
                               inView:(inout UIView **)view
{
    NSLog(@"%s", __PRETTY_FUNCTION__);
    // 显示在中心位置
    *rect = CGRectMake((CGRectGetWidth((*view).bounds)-2)*0.5f, (CGRectGetHeight((*view).bounds)-2)*0.5f, 2, 2);
}
```

<br><br/>
运行结果如下：<br><br/>

![](https://raw.githubusercontent.com/jixuqianxing/jixuqianxing.github.com/master/images/blogImages/20141128/20141128_3.gif)

至此，iOS 8下地UIActionSheet处理就完成了。在iOS 8下面使用这个还是有点小复杂。那么还有没有其他的方法呢？答案是肯定的，可以再本文开始的代码作如下处理：

```
- (IBAction)choosePhoto:(id)sender
{
    UIActionSheet *sheet = [[UIActionSheet alloc] initWithTitle:@"选择照片"
                                                       delegate:self
                                              cancelButtonTitle:@"取消"
                                         destructiveButtonTitle:@"照片库"
                                              otherButtonTitles:@"拍一张", nil];
    [sheet showInView:self.view];
}
// 替换- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex为下面这个方法
- (void)actionSheet:(UIActionSheet *)actionSheet didDismissWithButtonIndex:(NSInteger)buttonIndex
{
    if (buttonIndex == actionSheet.cancelButtonIndex) {
        return;
    }
    UIImagePickerController *picker = [[UIImagePickerController alloc]init];
    if (buttonIndex == 1) {
        [picker setSourceType:UIImagePickerControllerSourceTypePhotoLibrary];
    }else{
        [picker setSourceType:UIImagePickerControllerSourceTypeCamera];
    }
    [self presentViewController:picker animated:YES completion:nil];
}
```

这样也是可以解决前面的警告错误的。因为，现在是在didDismissWithButtonIndex:方法里面处理，也就是此时UIActionSheet已经从界面消失了之后再present UIImagePickerController。也就避免了同时present两个控制器。虽然，这种解决方法比较简单，但是苹果已经在iOS 8中废弃了UIActionSheet还有UIAlertView。所以，还是建议采用上面的解决方法。