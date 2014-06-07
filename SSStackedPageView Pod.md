SSStackedPageView : Intro to Pod and Reusable Cell Strategy 
----------------------------------

Have you used iOS 7 Passport or Reminder App? Isn't it rather nice to see the pages stack and flow with drags and taps? I ventured to create from scratch the same feature, using a single class I call SSStackedPageView. 

First the Repo: https://github.com/PlenipotentSS/SSStackedPageView

Feel free to walk along with me as I go through some of the more interested details.

![]() ![]()

First, the setup of the pod is quite easy, we need to setup the delegate in whatever controller we use and prepare the delegates:

```

#import "SSStackedPageView.h"
@interface ViewController () <SSStackedViewDelegate>

...
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    self.stackView.delegate = self;
    self.stackView.pagesHaveShadows = YES;
}

- (UIView *)stackView:(SSStackedPageView *)stackView pageForIndex:(NSInteger)index
{
   	//return the view for this index
}

- (NSInteger)numberOfPagesForStackView:(SSStackedPageView *)stackView
{
    //the total count of all views in the stack
}

- (void)stackView:(SSStackedPageView *)stackView selectedPageAtIndex:(NSInteger) index
{
   	//handler for page selection at index
}
```

That is all you need to get ready with the pod. Pretty great huh? There's some customizations that I'm sure I'll add as I proceed in using this in my other projects. Until then, we just have the option for shadows, and feel free to edit all the constants in the SSStackedPageView for further customizations. Namely those I currently use:

```
#define OFFSET_TOP 30.f
#define PAGE_PEAK 50.f
#define MINIMUM_SCALE 0.9f
#define TOP_OFFSET_HIDE 20.f
#define BOTTOM_OFFSET_HIDE 20.f
#define COLLAPSED_OFFSET 5.f
#define SHADOW_VECTOR CGSizeMake(0.f,-.5f)
#define SHADOW_ALPHA .3f
```

### Clip To Bounds!

A fun little part that I forgot my 2nd week of iOS programming was this awesome clipToBounds property on a CGLayer for a view. This does all that fun stuff that you see that allows a view to pass beyond the bounds of a given view. I used this a lot the last few days as I am developing UX features for the app Swing Local, as well as future apps I design.


###Reusing the cells/pages/views

Now the best thing about this implementation is the fact that I make use of the reusability of views so that the scrolling is faster, and uses the lazy loading practice. This in turn creates a more fluid experience and doesn't require as much active memory use as you scroll and interact with the stack. First and foremost, we use a scrollview to negotiate the offset for the display. We use this heavily in determining what views are visible, and what ones should be put to reuse.

The strategy behind this is knowing the visible range of elements that you have on screen. I originally built some hard coded examples on another project (https://github.com/PlenipotentSS/SSPagedView) that pages a current view and leaves peaks on either side to inform the user that scrolling is possible. After some testing, I created the following skeleton: (with annotations)

- 1) (starting in ```setPageAtOffset:```)Determine offset of current scroll area ```scrollView.contentOffset``` or minY of the current view if first load.

- 2) Determine the overlap for selection and find how many views can fit on screen at one time:

```
(PAGE_PEAK * i < end.y && PAGE_PEAK * (i + 1) >= end.y ) || i == [self.pages count]-1
```
where end.y is the point located at ```CGRectGetMaxY(view.bounds)``` or ```CGRectGetMaxY(view.bounds)+contentOffset```

- 3) make a NSRange starting with the first element visible to the last element visible. I typically keep this in an instance variable (here I used ```self.visiblePages```. The rest I need to store in the reusable queue for later use. This I do each time I add a new page the view (i.e. ```(void)scrollViewDidScroll:``` from the ```UIScrollViewDelegate``` delegate.

- 4) add the stacked views to the view:

```
- (void)setPageAtIndex:(NSInteger)index
{
    if (index >= 0 && index < [self.pages count]) {
        UIView *page = [self.pages objectAtIndex:index];
        if ((!page || (NSObject*)page == [NSNull null]) && self.delegate) {
            page = [self.delegate stackView:self pageForIndex:index];
            [self.pages replaceObjectAtIndex:index withObject:page];
            page.frame = CGRectMake(0.f, index * PAGE_PEAK, CGRectGetWidth(self.bounds), CGRectGetHeight(self.bounds));
            if (self.pagesHaveShadows) {
                [page.layer setShadowOpacity:SHADOW_ALPHA];
                [page.layer setShadowOffset:SHADOW_VECTOR];
                page.clipsToBounds = NO;
            }
            page.layer.zPosition = index;
        }
        
        if (![page superview]) {
            if ((index == 0 || [self.pages objectAtIndex:index-1] == [NSNull null]) && index+1 < [self.pages count]) {
                UIView *topPage = [self.pages objectAtIndex:index+1];
                [self.theScrollView insertSubview:page belowSubview:topPage];
            } else {
                [self.theScrollView addSubview:page];
            }
            page.tag = index;
        }
        
        if ([page.gestureRecognizers count] < 2) {
            UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(panned:)];
            [page addGestureRecognizer:pan];
            
            UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapped:)];
            [page addGestureRecognizer:tap];
        }
    }
}
```
Top to bottom, we see that this code ensures that there is no page currently in the array at this index, and we create a frame and appearance to how I want it. Here I have the y origin pushed out down by the current index. This creates that nice affect of looking like there is a uniform stack. Next I add shadows if the controller opts in for it. Then I add the page to the scroll view; note that I did some extra work noticing where I will insert the page. If it's coming in from the top, I need to ensure I keep the functionality of correct layering. Lastly, I need to add in the gesture recognizers to these views so that we can be informed when a drag or a tap has occurred on a view in our stack.


- 5) reload the visible cells to the screen (the fun animation part):

```
- (void)reloadVisiblePages
{
    NSInteger start = self.visiblePages.location;
    NSInteger stop = self.visiblePages.location + self.visiblePages.length;
    
    for (NSInteger i = start; i < stop; i++) {
        UIView *page = [self.pages objectAtIndex:i];
        
        if (i == 0 || [self.pages objectAtIndex:i-1] == [NSNull null]) {
            page.layer.transform = CATransform3DMakeScale(MINIMUM_SCALE, MINIMUM_SCALE, 1.f);
        } else{
            [UIView beginAnimations:@"stackScrolling" context:nil];
            [UIView setAnimationDuration:.4f];
            page.layer.transform = CATransform3DMakeScale(MINIMUM_SCALE, MINIMUM_SCALE, 1.f);
            [UIView commitAnimations];
        }
    }
}
```

This is the fun animation that we track for when load the view to the screen. You may be curious why we see the if statement containing: ```i == 0 || [self.pages objectAtIndex:i-1] == [NSNull null]```. This is purely an aesthetic addition, so that there is only a zoom effect when we scroll down, and we keep the stack look as we scroll up. Another funny change that I tried out is using ```+beginAnimations:context:``` and ```+commitAnimations``` combo for my animations. I wanted to see what other customizations I can do without using the ```+animateWithDuration:animations:```. I recall this is how we can create a bouncy affect that is ongoing if we ever wanted. Maybe more on that later =).

- 6) Reuse the cells!

As I passed over before, the reusable array is actual a queue. Remember data structures? A queue is simply a FIFO structure, which means an element that is First-In is also First-Out. Much like an actual "queue" in real life for those who do not say waiting-in-line. Here we reuse the cells indirectly by allowing the controller to maintain when we are going to reuse or if at all. (gotta love strict MVC patterns here). Let's take a look at that pageForIndex again:

```
- (UIView *)stackView:(SSStackedPageView *)stackView pageForIndex:(NSInteger)index
{
    UIView *thisView = [stackView dequeueReusablePage];
    if (!thisView) {
        thisView = [self.views objectAtIndex:index];
        thisView.backgroundColor = [UIColor getRandomColor];
        thisView.layer.cornerRadius = 5;
        thisView.layer.masksToBounds = YES;
    }
    return thisView;
}
```
This may look a little strange from the first run standpoint, because thisView will always be nil. Don't worry, because we resolve that the first time, and allow all other times to load from the reusuable queue (which is maintained by the SSStackedPageView itself).

- 7) Let it Roll!

We need to now just need to ensure that when we scroll we reload the pages. We started out setting up the setPageAtOffset, and then finished with reloading the visible pages. This is all we really need to do for our UIScrollViewDelegate:

```
- (void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    [self setPageAtOffset:scrollView.contentOffset];
    [self reloadVisiblePages];
}
```

Gotta love the simple code design =).

I hope that was helpful, and take a look at the cocoapod (hopefully soon to be released) 

Until then, Cheers!
