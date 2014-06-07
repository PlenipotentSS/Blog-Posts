Rotation Frame Persistence & Child View Controllers
---------------------------------------------------

I found an interesting occurrence this week while working on another app. My main target was to find the minimum requirement to ensure that layered view controllers maintain rotation when put inside a wrapping view controller. In my context, I wrote a split view controller for the iPhone (much like the UISplitController class for iPad). I wanted to customize some animations and behavior that I haven't found too simple in other open source projects. So My goal was to add these features without having to worry about 