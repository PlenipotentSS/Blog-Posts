Torch and NSOperationQueues!
----------

This is going to be a fun jump, or rather leap, into one of Apple's newest and soon to be most common threading API: NSOperationQueues and NSOperations. For those of you who will bitterly hold onto the Grand Central Dispatch (GCD), you should know that this new implementation as of (iOS6) is built on top of GCD. Therefore, if there is any optimization, it would be minimal and NSOperations will provide more readable code working with other developers. In this project we will be using a serialized Queue to queue a list of actions on top of each other and dequeue those action on a background thread. Now, you may be asking how I am activating the light on the background thread. Simply put, I'm not and with each operation block I add a block to call the main thread to update the UI. Let's take a look at some code.

```
#pragma mark - Transmit using many operations
- (IBAction)transmitString:(id)sender {
    if (self.torchQueue.operationCount == 0) {
        [[SSTorchAccess sharedManager] takeTorch];
        NSString *inputText = [NSString validateString:self.inputField.text];
        NSArray* morse = [NSString getSymbolsFromString:inputText];

        for (NSString* code in morse) {
            for (NSUInteger i=0;i<[code length];i++) {
                NSString *dotOrDash = [code substringWithRange:NSMakeRange(i, 1)];
                
                [self.torchQueue addOperationWithBlock:^{
                    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
                        unichar charString = [inputText characterAtIndex:counter];
                        [self updateTextLabelText: [NSString stringWithFormat:@"%c",charString] andMorseLabelText: code];
                        
                    }];
                }];
                if ([dotOrDash isEqualToString:@"."]) {
                    [self.torchQueue addOperationWithBlock:^{
                        [[SSTorchAccess sharedManager] engageTorch];
                        usleep(DOT_IN_MICROSEC);
                        [[SSTorchAccess sharedManager] disengageTorch];
                    }];
                } else if ([dotOrDash isEqualToString:@"_"]) {
                    [self.torchQueue addOperationWithBlock:^{
                        [[SSTorchAccess sharedManager] engageTorch];
                        usleep(DASH_IN_MICROSEC);
                        [[SSTorchAccess sharedManager] disengageTorch];
                    }];
                }
                [self.torchQueue addOperationWithBlock:^{
                    usleep(LETTER_DELAY_IN_MICROSEC);
                }];
            }
            [self.torchQueue addOperationWithBlock:^{
                usleep(WORD_DELAY_IN_MICROSEC);
            }];
        }
        [self.torchQueue addOperationWithBlock:^{
            [[SSTorchAccess sharedManager] releaseTorch];
        }];
    } else {
        [[SSTorchAccess sharedManager] releaseTorch];
        [self.torchQueue cancelAllOperations];
    }
}
```

There is the transmit method in its entirety. Now some may implement the background operations in one block call. This is possible, but in my implementation we can have a full queue of events lined up, and if the user ever cancels operations, it is very simple to cancel operations on the next thread cycle. This method is called on the transmit button that we mention in Part 1. Let's walk through some of the calls in this method.

First, check to see to see the count of the torchQueue operations (the background thread). If there is currently no transmission, then we start transmitting, otherwise we release the torch device and cancel all remaining operations.

If we star transmitting, we take the torch device and start looping through all the symbols we acquired in the string from part 1. Next we make some if:else statements depending on each symbol of the morse code word. We enqueue each operation in the background thread that engages the torch and disengages the torch according to the following rules:

####Rules:
* if symbol is dot then we have torch on for .1 seconds
* if symbol is dash then we have torch on for .3 seconds
* after each symbol we delay for .1 second
* lastly, we separate each morse word with .5 seconds.

Now, you may be thinking, this is a lot of operations in a thread - likely listing in the upper 100s. This doesn't slow your program down at all because it only runs one operation at a time. We do this in the initialization and setup of the NSOperationQueue:

```
self.torchQueue = [[NSOperationQueue alloc] init];
[self.torchQueue setMaxConcurrentOperationCount:1];
```

The last thing to examine here is the torch access. My implementation makes use of a singleton for the Device. Although Apple is leaning away from singletons, I am using it to find out if the device is currently transmitting (i.e. takeTorch and releaseTorch). I use this with a UIPageControl that allows you to switch between transmit and receive controllers.

New to singletons? Let's take a quick look at my singleton sharedManager:

```
+(SSTorchAccess*) sharedManager {
    static dispatch_once_t pred;
    static SSTorchAccess *shared;
    
    dispatch_once(&pred, ^{
        shared = [[SSTorchAccess alloc] init];
    });
    return shared;
}
```
This code is boiler plate, and ensures only one instantiation in any thread across the entire loop of the application. Let's take at those takeTorch and releaseTorch methods that limits operations to only transmit while transmitting.

```
-(void) takeTorch {
    if (!self.device) {
        self.device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    }
    _transmitting = YES;
}
-(void) releaseTorch {
    _transmitting = NO;
}

-(BOOL) isTransmitting {
    return self.transmitting;
}


```

Finally, activating the flash and disabling is very easy! In my implementation, I made two methods as follows:

```
-(void) engageTorch {
    if ([self.device hasFlash]) {
        [self.device lockForConfiguration:nil];
        [self.device setTorchMode:AVCaptureTorchModeOn];
        [self.device unlockForConfiguration];
    }
}

-(void) disengageTorch {
    if ([self.device hasTorch]) {
        [self.device lockForConfiguration:nil];
        [self.device setTorchMode:AVCaptureTorchModeOff];
        [self.device unlockForConfiguration];
    }
}
```

That's it for now! Stay tuned for the last explanation of the app and the github account to download the demo. I'll be implementing a receiver controller and some more UI styles and interactions for a better overall UX.

See you soon!
Steven


