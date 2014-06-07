Swing Local, Animating Table Cells, and more!
-------

I've ventured to create a fairly large and ambitious project, called Swing Local. I'll describe a little more in depth of what features I provide and how I provided them, but please take a look at my project: http://swinglocal.org. I'm looking for other developers as I expand this application across multiple platforms, but I mostly wanted to share some features that could inspire others in their ventures.

<table><tr><td>
{<1>}![](/content/images/2014/Feb/iOS_Simulator_Screen_shot_Feb_8__2014__6_35_26_PM.png)
</td><td>
{<3>}![](/content/images/2014/Feb/iOS_Simulator_Screen_shot_Feb_8__2014__6_35_50_PM.png)
</td></tr></table>
I have created a custom repository of calendars for swing dance venues around the world. I'm currently reaching out to as many organizers as possible, and am building a team to convince more swing dance communities to gather in this global initiative. With this repository, I'm building an API for other developers in the world to use the calendar's to do their own promotion and scene building. Let's talk about the iOS app itself.


####Feature 1: Animating Table View Cells

The first feature I'd like to showcase is the animation of table cells. As the user loads the table cell with data from the API I discussed, the cells fade and zoom in as they appear. This is a fun feature to add if your app needs a little more excitement than the stereotypical table views that you see everywhere. Now my goal was to display the cells from top to bottom with a fun animation that cascades as you view more. The easiest way I saw was to add a animation method within each call to: ```(UITableViewCell*)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath```. Thinking a little more into UITableView's, we know that as we scroll through a table view the above method is called for each new cell to be appeared. Other delegate methods are also called, but right now this is a sufficient place to call an animation. The method to add animation I'll call: ```(void) animateCell: (UITableViewCell*) cell```. The animation we want is going to contain a delay so that we allow time for each row to animate separately from others. You can play around with what timing works best for your app, but I'll use ```usleep(25000)``` for the fade in and ```usleep(40000)``` for the zoom look. For those interested why the large number and why it is called usleep(), then let's take a look at the name of the method. u is actually a symbol similar to the greek letter mew µ, and is a symbol for the measurement of units on the micro scale. Therefore µ = micro, and thus we measure sleep in microseconds. Done! 

Next we use a block animation from the UIView class method, and use two properties of the cell's layer: alpha and transform. Let's see the code and then comment a little more, but it should be fairly straight forward if you've used animateWithDuration and NSOperationQueues. Essentially, we will be using blocks, which is an essential feature of objective c and using them is a strong skill as most of Apple's framework uses blocks heavily.

```
#pragma mark - animation for cells
-(void) animateCell: (UITableViewCell*) cell
{
	[cell.contentView setAlpha:0.f];
    [self.cellOperationQueue addOperationWithBlock:^{
        usleep(25000);
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            [UIView animateWithDuration:.4f animations:^{
                [cell.contentView setAlpha:1.f];
            }];
        }];
    }];
    CATransform3D transform3d = CATransform3DMakeScale(1.15, 1.15, 1.15);
    cell.layer.transform = transform3d;
    [self.cellOperationQueue addOperationWithBlock:^{
        usleep(40000);
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            [UIView animateWithDuration:.4f animations:^{
                cell.layer.transform = CATransform3DIdentity;
            }];
        }];
    }];

}
```

Now the first thing to note is that I am using an instance variable of type NSOperationQueue to add the cell animation operations. This is important, because you don't want to call usleep() on the main thread as it will lock up the User Interface and your users will be unhappy if they wish to do anything else while your rows animate. Don't forget to instantiate the instance variable, as it is a one of the most common issues I run into while I debug my code:

```
self.cellOperationQueue = [NSOperationQueue new];
```

Now that we create an operation on the background thread, we need to make sure the animation occurs on the main thread, we do this by calling the mainQueue singleton property of NSOperationQueue. This may seem a little counterproductive, but if you think through it, the more you use the background thread to keep track of multiple actions, the better your design is. Now inside the mainQueue block we add the animation. We use the duration of .4f (.4 seconds in type float), because that is standard animation for many of Apple's designs. Then we change the alpha of the cell. Note we changed the alpha of the cell's contentView to 0.f previous to the block calls, and now we change it back to 1.f. The Alpha property is analogous to the opacity of an object. That's it for the fade affect. 

Next we transform the layer of the cell with the CATransform3D. Here we just want to scale it down to make it seem like it's falling into place. Just as we did the alpha property, we transform the layer from a 1:1.15 scale to a 1:1 scale. The 1:1 scale for the transform property is statically called as CATransform3DItentity.


####Feature 2: MKMapView Pins and Focus

Another fun feature is that I provide location reference to any venues in a given city. This is important, as you can see all the venue locations in a given date range for your saved cities. The way this app works is that it loads information to the map as each event is loaded from memory or the web. The call I designated the update to the map kit is: ```(void) updateMapPinForEvent: (Event*) event```. Let's take a look at the code and explain what is going on:

```
-(void) updateMapPinForEvent: (Event*) thisevent
{
    CLGeocoder *geoCoder = [[CLGeocoder alloc] init];
    [geoCoder geocodeAddressString:thisEvent.address completionHandler:^(NSArray *placemarks, NSError *error) {
        if (!error) {
            CLPlacemark *locationPlacemark = [placemarks lastObject];
            CLLocationCoordinate2D cityLocation = CLLocationCoordinate2DMake(locationPlacemark.location.coordinate.latitude,locationPlacemark.location.coordinate.longitude);
            EventAnnotation *thisAnnotation = [[EventAnnotation alloc] init];
            thisAnnotation.title = thisEvent.name;
            thisAnnotation.coordinate = cityLocation;
            
            [self.mapView thisAnnotation];
            if ([thisevent isLastEvent] {
            	[self.mapView showAnnotations:self.mapView.annotations animated:YES];
			}
        } else {
            // handle if no location was found
        }
    }];
}
```
You may be asking what CLGeocoder is and it has to do with MKMapview, but if you ever wanted to get a map pin without its longitude and lattitude, this is necessary to look it up in Apple's Geolocation system. Therefore the object method, ```geocodeAddressString:(NSString*) address completionHandler:^(NSArray *placemarks, NSError *error){} ``` requires a string that contains the address of your interested location. Then this goes into Apple's magic black box and comes back with an array of information in the given address (if one exists) and an error message (if no locational information is found). This is why you see (!error) for the first conditional statement inside the block. Each application will handle the error differently; you may want to tell the user, change the map view, or even pop the controller. I'll focus on the successful location lookup. 

Now the placemarks contain the longitude and latitude of the location you want to show on the map. You may want to look at more what the array of placemarks contain, but for now we will just extract the longitude and latitude. We then create a CCLocationCoordinate2D (a long word for a longitude and latitude pin on the globe) with the location placemark. Next you may notice we are creating an EventAnnotation object, which is a custom sub-classed object that you can see below:


```
#import <Foundation/Foundation.h>
#import <MapKit/MapKit.h>


@interface EventAnnotation : NSObject <MKAnnotation>

@property (nonatomic, copy) NSString *title;
@property (nonatomic, assign) CLLocationCoordinate2D coordinate;

@end
```

Now with any object that you conform with the protocol MKAnnotation, you get a title and a coordinate property that you must set to make it work with Apple's Map Kit. Forwarding the code from before, let's look at annotations and adding them to the Map View:

```
EventAnnotation *thisAnnotation = [[EventAnnotation alloc] init];
thisAnnotation.title = thisEvent.name;
thisAnnotation.coordinate = cityLocation;
            
[self.mapView addAnnotation:thisAnnotation];
if ([thisevent isLastEvent] {
	[self.mapView showAnnotations:self.mapView.annotations animated:YES];
}
```

The first three lines are self explanatory, but for reference, the title allows you to type the pin and it will show you that pin's title. The coordinate is where on the map it should be placed. You then want to add this annotation to the map by the object method: ```addAnnotation:(id < MKAnnotation >)annotation```. This then adds a pin to the map, but what happens if you have lots of pins and they may be out of the scope of the current map view? Well that is why we have the last conditional block. If we are looking at the last event, then we will want to make it so that the map view focuses on all the annotations. Now This method hasn't worked 100% for me in the past, but without having to write more math than code, this is the easiest way to do it.

If you are interested in the math however, it's a fairly simple process. You have to find the best square with a given edge padding so that it shows all the pins. This is essentially finding the largest rectangular distance among all the pins and use the pythagorean theorem ```(a^2+b^2=c^2)``` where the rectangular distance is ```c^2```. We then have to create the sides with a distance of ```(2x^2)^(1/2) = c```, where x is the side and c is the hypotenuse. You have to convert the x in the correct Meters per Mile to display in the Map View - but that is the start at least!

Enjoy, and Cheers!
