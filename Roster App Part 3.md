Roster App Day 3
-------

UIImagePickerController! For those not aware, Apple is making it increasingly easier to reach all their devices' hardware with easy pre-made APIs. UIImagePickerController is exactly that! Given choices through a UIActionSheet, we find ourselves choosing between the Photo Library or the camera itself to select images from.

Things I learned
------------
Objective-C is by far the longest variable Constant names ever. The type to compare the image controller source is


```
[UIImagePickerController availableMediaTypesForSourceType:UIImagePickerControllerSourceTypeCamera]
```

That's it. Then all you have to do is present the image controller and have a method from the UIImagePickerControllerDelegate (after the image is take) or acquiring ALAuthorization for the Photo Library.

For use of this project, the imagePickerController:didFinishPickingMediaWithInfo: is below: 

```
-(void)imagePickerController:(UIImagePickerController*) picker didFinishPickingMediaWithInfo:(NSDictionary *)info {
    [picker dismissViewControllerAnimated:YES completion:^{
        UIImage *studentImage = [info objectForKey:@"UIImagePickerControllerEditedImage"];
        self.placeHolder.image = studentImage;
        [self saveStudentImage:studentImage];
        [self.imageButton setTitle:@"" forState:UIControlStateNormal];
    }];
}
```

where saveStudentImage saves not only the image to the Documents directory of the app, but also stores the information to the student object in our plist.

To load the Photo Library, you have a few calls to verify authorization, again the code below explains that process:

```
[self presentViewController:myPick animated:YES completion:^{
        if (myPick.sourceType == UIImagePickerControllerSourceTypePhotoLibrary) {
            ALAuthorizationStatus status = [ALAssetsLibrary authorizationStatus];
            if (status != ALAuthorizationStatusNotDetermined && status != ALAuthorizationStatusAuthorized) {
                [myPick dismissViewControllerAnimated:YES completion:^{
                    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Photo Access Denied" message:@"Please Provide Access in your Settings. In Settings, enable Roster App in Privacy->Photos" delegate:self cancelButtonTitle:@"Okay" otherButtonTitles:nil];
                    [alert show];
                }];
            }
        }
    }];
```

That's it! Yet again Apple, you make my life too easy =)!    
