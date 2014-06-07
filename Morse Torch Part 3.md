HUD ++
----------
Saving the UI result for the last part of this demo object, let's talk about subclassing a pod and adding some further improvements to your UX. In this example, I'll be using a linear color change from my color category class that defines a red (cancel color) and green (transmit color). I use M13ProgressSuite's progress ring and change the color from red to green depending on the progress rate.

Subclassing is as easy as creating a new file, and selecting which class you want to inherit from. Xcode's interface makes life so easy sometimes, you often forget about the days of rewriting the entire file from scratch in a notepad.

After subclassing M13ProgressViewRing, all I had to do was override the setProgress:animated: method.

```
-(void)setProgress:(CGFloat)progress animated:(BOOL)animated {
    [super setProgress:progress animated:animated];
    UIColor *thisProgressColor =[self getNextColor:progress];
    [super setPrimaryColor:thisProgressColor];
    [super setSecondaryColor:thisProgressColor];
}

-(UIColor*) getNextColor:(CGFloat) progress {
    CGFloat r1,g1,b1,a1;
    [[UIColor getTransmitBgColor] getRed:&r1 green:&g1 blue:&b1 alpha:&a1];
    
    CGFloat r2,g2,b2,a2;
    [[UIColor getCancelBgColor] getRed:&r2 green:&g2 blue:&b2 alpha:&a2];
    
    CGFloat r,g,b,a;
    r = r2-progress*(r2-r1);
    g = g2-progress*(g2-g1);
    b = b2-progress*(b2-b1);
    a = a2-progress*(a2-a1);
    
    return [UIColor colorWithRed:r green:g blue:b alpha:a];
}
```

Now there is also another way to refactor this, that would involve acquiring the UIColor from the UIColor category that would normalize the cancel color and green color vs. having to find the linear progression for each r,g,b, and a variables. That's soon to come I'm sure.

Last in store is the Receive controller! We will be using a great open source project at https://github.com/zuckerbreizh/CFMagicEventsDemo to acknowledge the brightness of the flash. If I have time, I'll subclass his/her project or create one of my own that would normalize (much like the colors) the image feed and would then cancel out the majority of noise to better read the morse code. Who knows, this may even reach the app store if I can't find anything else that is better! For now, it's just a fun open source demo project!

Almost There!