Cocoa Pods
----------

Those not knowing about Cocoa Pods, need to go there right now! Seriously, right now! Cocoa Pods is an amazing tool for iOS developers that enable pre-made features that you can import into your project. Most of them are licensed under the MIT software license, and often acknowledging is the only thing that you should do when using these pods. Getting cocoa pods is slightly difficult, especially if you don't have ruby installed already. Cocoa Pods has some awesome documentation, but let's also walk through getting it setup here. Assuming you do have ruby, you just go to your terminal and type:

```
sudo gem install cocoapods
```

That will download the cocoapods gem and next is just setting up cocoa pods for your run.

```
[sudo] gem install cocoapods
```

After that you are ready to start looking at some pods. If you find a pod you would like, you need to go to your project folder and create a file titled (no extension): Podfile .

In this Podfile, you add the code:

```
platform :ios '7.0'

pod POD_NAME POD_VERSION
```

the POD_NAME and POD_VERSION can be found in the cocoapod, and is easier to copy and paste.

That's it for Cocoa pods, and expect more info on the morse code app that will be making use of the M13ProgressSuite Hub display. 

Cheers,
Steven
