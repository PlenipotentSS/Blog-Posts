Potential Cocoa Pod: SplitPeakViewController !
---------------
I'm submitting my version a split view controller that I created a week ago. I wanted to share it, because of how much I've enjoyed working on it (HA!). Seriously though, I found quite a few of them that were a little to detailed than they needed to be. I wanted to provide a simple setup, and allowing more freedom for other users to change the pod to how they want. Take a look at my project below:

```
https://github.com/PlenipotentSS/SplitPeekViewController
``` 

The only reason why I really wanted to push this out in cocoapods/controls, because I've ran into a few autorotation issues with a lot of pods. This is an easy interface that just uses a front and back identifier to tie your split controller into your storyboard. Most is done programmatically, but leaves room to customize given storyboard or nib files. Check out the demo project that I included with it. I've subclassed this project twice now, and everything works great! If you check out auto rotation, frames and layered view controllers I go into more detail in the search to not recode my issues in rotation methods. Not saying that this is a bad idea, but I do feel that if the issue has to be resolved in more and more cases, then it may not be that scalable...

My friend made an awesome find in the search of making your view controller rotate correctly. His site is: ```http://bytemeetsworld.com/rss```. He has some great insight in many of the "buggier" features in iOS and Xcode