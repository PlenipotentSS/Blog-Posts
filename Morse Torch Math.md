Brightness, Normalization and AVCaptureAudioDataOutputSampleBuffer Delegate
---------

Let's talk about some math... Or rather, let's talk about the Morse Torch Demo and using math to provide a better detector for brightness changes. The Github project that was suggested has the correct methodology, but assumes too much to provide accurate information. His project can be found at: ``` https://github.com/zuckerbreizh/CFMagicEventsDemo``` Let's walk through how he uses a buffer first.

The project uses a delegate from ```AVCaptureAudioDataOutputSampleBuffer``` (phew), where we will be acquiring an image buffer. The method used is:```(void)captureOutput:didOutputSampleBuffer:fromConnection```. (phew again)... The image buffer is then called through a ```AVCaptureSession```.

```
AVCaptureDevice *captureDevice = [self getBackCamera];
	
AVCaptureDeviceInput *videoInput = [AVCaptureDeviceInput deviceInputWithDevice:captureDevice error:&error];
_captureSession = [[AVCaptureSession alloc] init];
    
_captureSession.sessionPreset = AVCaptureSessionPresetLow;
    
[_captureSession addInput:videoInput];
```

To make this work as planned, there is also some settings that are required:

```
AVCaptureVideoDataOutput *videoDataOutput = [[AVCaptureVideoDataOutput alloc] init];
    
//  pixel buffer format
NSDictionary *settings = [[NSDictionary alloc] initWithObjectsAndKeys: [NSNumber numberWithUnsignedInt:kCVPixelFormatType_32BGRA], kCVPixelBufferPixelFormatTypeKey, nil];

videoDataOutput.videoSettings = settings;
    
dispatch_queue_t queue = dispatch_queue_create(QUEUE_NAME, NULL);

[videoDataOutput setSampleBufferDelegate:(id)self queue:queue];
[_captureSession addOutput:videoDataOutput];
```

the most important customization is the settings that tells how we will be configuring the buffer, which is stated with the NSNumber: ```numberWithUnsignedInt:kCVPixelFormatType_32BGRA``` attached to the enum key: ```kCVPixelBufferPixelFormatTypeKey```. This will help us get the pixel base through the image buffer. Now we can talk about how we will get the pixel information in the ```AVCaptureAudioDataOutputSampleBuffer``` protocol method.

```
- (void)captureOutput:(AVCaptureOutput *)captureOutput
didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer
       fromConnection:(AVCaptureConnection *)connection
{
	CVImageBufferRef imageBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);
   if (CVPixelBufferLockBaseAddress(imageBuffer, 0) == kCVReturnSuccess){
        UInt8 *base = (UInt8 *)CVPixelBufferGetBaseAddress(imageBuffer);
        
		...
	}
}
```

Walking through this, we acquire the image buffer from the sample buffer (alternatively we can also get audio buffer) and acquire the base address for the given image buffer. Now that we have a base, we can get the width,height and bytesPerRow to loop through and get the RGB values. To get that information, let's look at how we get the RGB values.

```
size_t bytesPerRow      = CVPixelBufferGetBytesPerRow(imageBuffer);
size_t width            = CVPixelBufferGetWidth(imageBuffer);
size_t height           = CVPixelBufferGetHeight(imageBuffer);
UInt32 totalBrightness  = 0;

for (UInt8 *rowStart = base; counter_row < height; rowStart += bytesPerRow, counter_row++){
	int counter_column = 0;
	for (UInt8 *p = rowStart; counter_column<width; p += 4, counter_column++){
		UInt32 red = p[0];
		UInt32 blue = p[1];
		UInt32 green = p[2];
		UInt32 thisBrightness = (.299*r + .587*g + .114*b);
		
		...
	}
}

```

Now you can see that we are looping through the matrix of pixels within the given buffer, and we store these in red, blue and green values. A few details: Why are we storing the brightness with r,g and b values by multiplying the constants .299, .587 and .116? And why are we using UInt8 (p) and UInt32 ?

First, those constants when used with R,G and B values provide the Luma of an image (http://en.wikipedia.org/wiki/Luma%5F%28video%29). This is in effort to create a better sampling to determine actual brightness and removing color saturation. Typically you use Luma with Chroma, and that is why we use p+=4 (4:1:1). 

Second, we use UIn8 because there is 8-bit integer, and similarly we have to increase the number of bits to store the integer, so we upgrade it to an unsigned integer with 32-bit. 

Now, the Math!
--------------

The github project accumulated just the brightness by keeping track of the total brightness by: ```UInt32 brightnessTotal +=thisBrightness```. As you may see, storing just the total brightness across all pixels may create extra noise than necessary. You can attempt to normalize the total brightness, but it doesn't create a strong threshold to detect a strong spike of brightness. Instead, I calibrated the current buffer by averaging the brightness at each pixel. Then by using a last brightness accounting, I normalized the total brightness to keep the brightness at each iteration  to approximately 100. I then added a sensitivity slider to alter the threshold. The sensitivity accounts for distance and environment light. 

With a calibration feature, I had a better ability to maintain a basis for the surroundings, and then found that the threshold needed not be incredible large. I found that in mid-light settings, a threshold of only about 103 was required, whereas the darker the room, the higher threshold was better likely due to aperture changes. The below code is the meat of the detection object, then sending NSNotifications to the receiver controller:

```
int thisBrightness = [self calculateLevelOfBrightness:totalBrightness];
if( thisBrightness > MIN_BRIGHTNESS_THRESHOLD ){
                
	if(thisBrightness > self.brightnessThreshold ) {
                    
		//check to how long we are above threshold & recalibrate if needed
 		if (self.timeBeyondThreshold == 0.f) {
                        
			self.timeBeyondThreshold = [NSDate timeIntervalSinceReferenceDate];
		}
		if ([NSDate timeIntervalSinceReferenceDate]-self.timeBeyondThreshold > 2.f) {
                        
			//recalibrate
			self.brightnessMatrix = [NSMutableArray new];
  			[self.calibrationNumbers removeAllObjects];
		} else {
                        
			//send Light ON notification
       	[[NSNotificationCenter defaultCenter] postNotificationName:@"OnReceiveLightDetected" object:nil];
		}
	} else {
                    
       //send Light OFF notification
		[[NSNotificationCenter defaultCenter] postNotificationName:@"OnReceiveLightNotDetected" object:nil];
		self.lastTotalBrightnessValue = totalBrightness;
		self.timeBeyondThreshold = 0.f;
	}
}else{
                
	//send Light OFF notification
	[[NSNotificationCenter defaultCenter] postNotificationName:@"OnReceiveLightNotDetected" object:nil];
	self.lastTotalBrightnessValue = totalBrightness;
	self.timeBeyondThreshold = 0.f;
}
```

And that's it! I am not showing the receiver method here, as verifying the time signatures for the length of light hinders primarily on the light detector and not the receiver controller. Go MVC!

Steven