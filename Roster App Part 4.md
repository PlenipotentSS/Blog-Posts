Roster App Day 4
-------

A quick note about UIScrollViews... If you're ever having trouble working with scroll views, there's a good chance that you've confused how exactly to populate the view. Scroll view is not the easiest thing to plan and execute in just the Storyboard UI. Think a little bit about where your subviews will be put, and don't be afraid to pull out some pen and paper and plan out on your own before venturing into putting things in the scroll view. 

You'll see the 4" screen given in the Storyboard editor is a little small for what you may want. I guess that means we're putting those bad boys in programmatically (my least favorite word ever, just shy of existing-state).

```
[self.rSlider setThumbImage:[UIImage imageNamed:@"RSliderButton.png"] forState:UIControlStateNormal];
    [self.gSlider setThumbImage:[UIImage imageNamed:@"GSliderButton.png"] forState:UIControlStateNormal];
    [self.bSlider setThumbImage:[UIImage imageNamed:@"BSliderButton.png"] forState:UIControlStateNormal];
    [self.rSlider addTarget:self action:@selector(updateBackgroundColor) forControlEvents:UIControlEventValueChanged];
    [self.gSlider addTarget:self action:@selector(updateBackgroundColor) forControlEvents:UIControlEventValueChanged];
    [self.bSlider addTarget:self action:@selector(updateBackgroundColor) forControlEvents:UIControlEventValueChanged];
    
self.scrollView.contentSize = CGSizeMake(self.view.frame.size.width, (self.bSlider.frame.origin.y+self.bSlider.frame.size.height+20));
    self.scrollView.delegate = self;
    self.scrollView.userInteractionEnabled = YES;
```
    
Expecting it more difficult than that? Of course, UI additions in code make things lengthy and difficult to read, but it's really the best way to use UIScrollView if you want to have a long list of items on the view. Alternatively, you can design individual UITableViewCells and just use a UITableView. It's really the preference for this level as how the developer wants to display his information. As subclassing and future iterations of, in this example, detail views. It may be more advantageous for a scroll view, as you can always increase the size by keeping track of the lowest subview that you may add. It saves the time subclassing yet another cell.

Sliders
------

Not much to say, as sliders are just a fun way to keep track of a user input from a custom made scale. Since this app designs three sliders for the values of R,G,B, we use the values between 0-1. Much like other inputs, we can manage any value change of the sliders (see above), and handle it in our own selector updateBackgroundColor.

```
-(void)updateBackgroundColor {
    self.view.backgroundColor = [UIColor colorWithRed:(self.rSlider.value)
                                                green:(self.gSlider.value)
                                                 blue:(self.bSlider.value)
                                                alpha:1];
    [self.student setRGB:[[NSArray alloc] initWithObjects:[NSNumber numberWithFloat:self.rSlider.value],[NSNumber numberWithFloat:self.gSlider.value],[NSNumber numberWithFloat:self.bSlider.value], nil]];
    UIColor *textColor = [UIColor colorWithRed:(1-self.rSlider.value)
                                     green:(1-self.gSlider.value)
                                      blue:(1-self.bSlider.value)
                                     alpha:7];
    [self refreshTextViews:textColor];
}
```

As much of this is boiler plate, I am keeping track for those interested, or even myself on how you may want to implement a UISlider.

Cheers,
Steven
