##Core Animation Demo

Core Animation is the main animation and graphics rendering infrastructure built into iOS. Unlike many Apple frameworks, Core Animation along with Core Graphics sit below UIKit/AppKit. This means that it is included for each controller that inherits UIKit. This also means that it is tightly integrated into the view workflows as well as many of the built in features.

First, feel free to follow along the github project: https://github.com/PlenipotentSS/Core-Animation-Demo.

Have you ever used UIView's  ```animateWithDuration:animations:completion```? Well you're using Core Animation and may not know about it. This is very common feature used in iOS Programming, but I'll go a little deeper into Core Animation's customization. To show a more in depth usage of Core Animation, we are going to be taking the SplitViewController design and change a simple translation animation to a more complex animation involving scaling AND translation. 

This can be done several different ways, but the main reason why we have to go deeper into Core Animation is because using UIView's ```animateWithDuration:animations:completion``` can only animate one property from the same element at a time. To do this, we drop down the Core Animation layer and add ```CABasicAnimations```. 

We will be changing one of the methods that show the back menu: ```(void)showMenuSplit```.

The main code that we will be looking to change is below: 

```
    [UIView animateWithDuration:.4f animations:^{
        self.frontViewController.view.frame= newFrontFrame;
        self.backViewController.view.frame = [self getBackViewRectOpen];
    }];
```

Here is a short road map of what we are looking to do with Core Animation:

 - Use CABasicAnimation with KeyPath
 - add customization such as ```fromValue```, ```toValue``` and ```duration```.
 - We apply the animations, and set the completed transforms and frames.
 
#####Position

The following Code tells the front view controller to open for the menu: 

```
/// 1
    CABasicAnimation *frontPosAnim = [CABasicAnimation animationWithKeyPath:@"position"];
    

/// 2
    frontPosAnim.fromValue = [NSValue valueWithCGPoint:self.frontViewController.view.layer.position ];
    frontPosAnim.toValue = [NSValue valueWithCGPoint:newPosition];
    frontPosAnim.duration = .4f;    
    

/// 3
    frontPosAnim.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    
```

First odd thing you'll notice is the ```animationWithKeyPath``` (1). This utilizes the Key-Value-Coding (KVC) programming that iOS uses to send messages throughout its apps. More information on KVC can be found here: https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html

Next we set setup the animation parameters: ```fromValue```, ```toValue```, and ```duration```. Notice that fromValue and toValue are both CGPoints. In this situation, we use them as the center points for the layers. The duration is the standard duration of .4. 

Then, we look at the timing function and the standard EaseInEaseOut timing function. There is more information on creating custom animation functions, but for more information read https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Animation_Types_Timing/Articles/Timing.html.

We duplicate this process to the translation (position) of the back controller, as well as the scaling effect. The following code should look fairly familiar now: 

```
    CABasicAnimation *frontPosAnim = [CABasicAnimation animationWithKeyPath:@"position"];
    
    CGPoint newPosition = self.frontViewController.view.layer.position;
    newPosition.x = newXOrigin+CGRectGetWidth(self.frontViewController.view.frame)/2;
    
    frontPosAnim.fromValue = [NSValue valueWithCGPoint:self.frontViewController.view.layer.position ];
    frontPosAnim.toValue = [NSValue valueWithCGPoint:newPosition];
    frontPosAnim.duration = .4f;
    frontPosAnim.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    
    CABasicAnimation *backPosAnim = [CABasicAnimation animationWithKeyPath:@"position"];
    
    CGPoint newPositionBack = self.frontViewController.view.layer.position;
    newPosition.x = 0.f;
    
    backPosAnim.fromValue = [NSValue valueWithCGPoint:self.frontViewController.view.layer.position ];
    backPosAnim.toValue = [NSValue valueWithCGPoint:newPositionBack];
    backPosAnim.duration = .4f;
    backPosAnim.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
    
    CABasicAnimation *scale = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
    scale.fromValue = @0.75; // Your from value (not obvious from the question)
    scale.toValue = @1.0;
    scale.duration = 0.4;
    scale.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];

```

All that remains is adding the animations to the layers and setting the values. To add the animations, we must add them to the layer of the views. This is simply done by messaging ```addAnimation:forKey:``` for the layer object. Lastly, we update the views to apply the layering affects after the animation. This step is very important!

```
    [self.backViewController.view.layer addAnimation:scale forKey:@"move forward by scaling"];
    [self.frontViewController.view.layer addAnimation:frontPosAnim forKey:@"position"];
    [self.backViewController.view.layer addAnimation:backPosAnim forKey:@"position"];
    
    // Set end value (animation won't apply the value to the model)
    self.backViewController.view.transform = CGAffineTransformIdentity;
    self.frontViewController.view.frame= newFrontFrame;
    self.backViewController.view.frame = [self getBackViewRectOpen];
```

That is it! This is just a taste on how going further into Core Frameworks can add elegant complexity to your apps. 

Enjoy!