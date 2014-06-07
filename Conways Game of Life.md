TANGENT PROJECT!
---------------------
A fun little project that was thrown to us last minute. Here we create a UI for Conway's Great Game of Life. If you're unfamiliar, type "Conway's Great Game of Life" into Google and you'll see a fun graphic in the far right side that show's boxes traveling in seemingly random order. Well, surprise! They are not random. John Horton Conway was a british Mathematician who designed a world of boxes with the following rules:

####The Rules:

* A box with Fewer than 2 neighbors dies (Loneliness)
* A live box with > 3 live neighbors (Overpopulation)
* A live box with 2 or 3 Lives (enough to keep alive)
* A dead box with exactly 3 neighbors is born (takes 3 to tango)

Then as the user inputs the initial configuration of boxes, the game is then played to see how the boxes die and live over time. A fun side note that began around this time was the idea of 'Simulated Games' based off correlations to mathematical models simulating life's rise and falls of societies. Now as most of this surprise project is UI based, I will highlight a couple interesting findings when designing this app.

Most games have a run loop for when the game is running, and using a while loop can be often found with a flag triggering the end. Doing this, one must be careful to ensure that the application isn't locking in the while loop as that's a big NO-NO when hoping for any star rating. In this project, I used an NSTimer that continued to run and refresh the UI every .1 seconds. A short note, do not use the class method timerWithTimeInterval:target:selector:userInfo:repeats:] . This is essentially creating a timer with a time interval. It does not actually create and run the timer. I mention this because I auto-tabbed to much and assumed I was using the correct method for way too long in my debugging. The proper method is the following:

```
self.runTimer = [NSTimer scheduledTimerWithTimeInterval:.1 target:self selector:@selector(runStateChange) userInfo:nil repeats:YES];
[self.runTimer fire]; 
```

In my implementation, I use runStateChange to process each generation of the "Life Cycle of Boxes." Therefore, calling this Timer and firing ensures the method to run every .1 seconds. Lastly, let's see how we can cancel this timer when running: 

```
-(void) stopGameLoop {
    [self.delegate setGridState:YES];
    [self.runTimer invalidate];
    self.runTimer = nil;
}
```
the main method to cancel the timer is [TIMER_INSTANCE invalidate]. That's it for timers, but if you're interested to know about the UI interactions, check out the gitBo (my accidental terminology of github Repo): https://github.com/PlenipotentSS/ConwaysGameOfLife

Also check out the project at CodeFellows: https://www.codefellows.org/blogs/the-game-of-life
