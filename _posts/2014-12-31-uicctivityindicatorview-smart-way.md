---
layout: post
title: Adding UIActivityIndicatorView the Smart Way
---

As iOS developer we often do some synchronous process and use a loading indicator to a view to tell user that loading is in progress. There are some faboulous libraries out there to show a loading is in progress, like [SVProgressHUD][SV] and [MBProgressHUD][MB].

But somewhat we just want a simple UIActivityIndicatorView to show that process. Normally we add it by using this simple code:

```objective-c
UIView *superview = self.view;

CGFloat activityIndicatorHeight = 20.0;
CGFloat activityIndicatorWidth = 20.0;

CGFloat originX = superview.frame.size.width/2 - activityIndicatorWidth / 2;
CGFloat originY = superview.frame.size.height/2 - activityIndicatorHeight / 2;

UIActivityIndicatorView *act =[UIActivityIndicatorView new];
act.backgroundColor = [UIColor redColor];
act.frame = (CGRect){originX,originY,activityIndicatorWidth,activityIndicatorHeight};
[self.view addSubview:act];

[act startAnimating];
```

Phew, for showing a simple UIActivityIndicatorView on the center of the screen we need at least this much of code. Imagine if we want to add it to a dozens of cells on a UICollectionView. Or even on a different view on different view controller. Soon your view controllers or custom views become so big and not easy to maintain.

Now, with several steps that I will show you we can show a simple UIActivityIndicatorView by just this much cleaner command:

```objective-c
[superview.activityIndicatorView startAnimating];
```
and hide it using this command:

```objective-c
[superview.activityIndicatorView stopAnimating];
```

Much simple yeah? So how do we do that? We will use these 2 Objective C features:

1. [Categories][CAT]
2. [Associated Objects][ASS]

We use category for placing our activity indicator initialization so that we have a clean and better separation of concern to our code, then since we can't have properties in a category, we use associated object to add custom properties to UIView class.

First, we create a UIView category named UIView+ActivityIndicator and add a property in its header.

```objective-c
#import <UIKit/UIKit.h>

@interface UIView (ActivityIndicator)

@property(nonatomic, strong) UIActivityIndicatorView *activityIndicatorView;

@end
```
Since we use runtime library of Objective C, we need to import it beforehand

```objective-c
#import <objc/runtime.h>
```

then we add getter to that property in the implementation file

```objective-c
- (UIActivityIndicatorView *)activityIndicatorView
{
    UIActivityIndicatorView *activityIndicatorView = objc_getAssociatedObject(self, @selector(activityIndicatorView));
    
    if (!activityIndicatorView) {
        
        CGSize loadingSize = CGSizeMake(20, 20);
        CGFloat activityIndicatorOriginX = self.frame.size.width/2 - loadingSize.width/2;
        CGFloat activityIndicatorOriginY = self.frame.size.height/2 - loadingSize.height/2;
        
        activityIndicatorView = [[UIActivityIndicatorView alloc]initWithFrame:
                                 CGRectMake(activityIndicatorOriginX,
                                            activityIndicatorOriginY,
                                            loadingSize.width,
                                            loadingSize.height)];
        
        [activityIndicatorView setActivityIndicatorViewStyle:UIActivityIndicatorViewStyleGray];
        [self addSubview:activityIndicatorView];
        
    }
    
    // default is stop the animation
    [activityIndicatorView stopAnimating];
    [self setActivityIndicatorView:activityIndicatorView];
    
    return activityIndicatorView;
}
```

the getter will create a new default object of UIActivityIndicatorView, if there's none, before adding it to the view.

Voila!

Now, by importing our category file `"UIView+ActivityIndicator.h"`, you can `startAnimating` your UIActivityIndicatorView right at the center of the view.

```objective-c
[view.activityIndicatorView startAnimating];
```

You can see full example in [Github][EX]

[SV]:https://github.com/TransitApp/SVProgressHUD
[MB]:https://github.com/jdg/MBProgressHUD
[ASS]:http://nshipster.com/associated-objects/
[CAT]:http://rypress.com/tutorials/objective-c/categories
[EX]:https://github.com/mamaz/MMZHelpers/tree/master/UIView%2BHelpers
