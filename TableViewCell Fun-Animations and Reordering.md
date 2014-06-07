UITableViewCell Editing and Sorting
---------------------------------------------

Ever wanted to see your table view animate a little as you scroll? Or how about changing the order with UITableView's built in editing methods? I'll try to go over as much as I can with these features, but realistically this will be just a taste of what is possible with UITableViews. I won't go over the basics of UITableViews, as they are plenty of well explained tutorials. However, I am always looking for more integration features that are already made and can be made within a UITableView structure.

A few things that I started with, I sub classed UITableView cell, as well as looked up UITableView's documentation to really see what power it holds. I'll first talk about editing the table cells. UITableView has built in reordering, editing, deleting and adding. (what!?) Yep, you don't have to create any custom code, you just have to use Apple's methods and you update the source data. Using a table view in a ```UIView``` requires the class to conform to the UITableViewDataSource. UITableViewDelegate is a suggested protocol to conform to, but is especially required to edit the table cells. Also, Any UIButton can begin a table view's editing. Let's take a look at how you may start a UIViewController class to have the ability to edit a UITableView's cells:

```
#import "SomeUIViewController"
@interface SomeUIViewController()

@property (weak, nonatomic) IBOutlet UITableView * theTableView;
@property (strong, nonatomic) NSArray * data;

@end

@implementation SomeUIViewController <UITableViewDataSource, UITableViewDelegate>

-(void)viewDidLoad
{
	[super viewDidLoad];
	
	//required to populate table view
	self.theTableView.dataSource = self;

	//contains many helpful methods, such as editing cells
	self.theTableView.delegate = self;
}

-(IBAction)editButtonPushed:(id) sender
{
	if ( [self.theTableView isEditing] )
	{
		[self.thetableView setEditing:NO animated:YES];
	} else 
	{
		[self.thetableView setEditing:YES animated:YES];
	}
}

```

####Cell Editing Setup

The first method allows you to define what sort of editing style while editing the table view. The types are: 

```
- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath
{
	return UITableViewCellEditingStyleNone;
}
```

Where the editing styles are:

```UITableViewCellEditingStyleNone``` - cell at the indexPath has no editing cabilities (this is the default method), 
```UITableViewCellEditingStyleDelete``` - This allows you to delete the cell at the indexPath (marked by a red deletion symbol),
``UITableViewCellEditingStyleInsert`` - In place of a deletion cell, this contains a green symbol marking it to be added to the current table view. 

Next method tells the tableview what cells are allowed to be edited. In this example, we tell the table view that all cells can edit at a given index path. You may want to limit the cells by an indexPath who can be edited (especially if you make cells to be specifically different than other populated cells).

```
- (BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath
{
	return YES;
}
```

Now that we have defined the editable cells, we can look at what cells can be reordered. This is as simple as tell what indexPaths can move. In this example, we can move any cell except the first in each section:

```
- (BOOL)tableView:(UITableView *)tableview canMoveRowAtIndexPath:(NSIndexPath *)indexPath
{
	if ( indexPathPath.row == 0 )
	{
		return NO;
	}
	return YES;
}
```

####Saving Actions in Editing Session

If you've tried this so far, you may have noticed that all the elements can be removed, moved and added, but not saved at the end of editing the cells. That is because, we are only manipulating the table view cells, and not the actual data that populate those cells (the array: ```self.data```). Let's take a look at how to commit deletion of cells (as the process is no different with others):

```
- (void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView beginUpdates];
    if (editingStyle == UITableViewCellEditingStyleDelete) {
    	[self.data removeObjectAtIndex:indexPath.row];
        [tableView deleteRowsAtIndexPaths:[NSArray arrayWithObjects:indexPath, nil] withRowAnimation:UITableViewRowAnimationFade];
    }
    [tableView endUpdates];
}
```

As you can see we remove the object as well as confirm the deletion.

Next we need to confirm the reorder of the cells. We similarly need to update the data array, but do not need to change the table view cells.

```
-(void)tableView:(UITableView *)tableView moveRowAtIndexPath:(NSIndexPath *)sourceIndexPath toIndexPath:(NSIndexPath *)
{
    id dataObject = [self.data objectAtIndex:sourceIndexPath.row];
    [self.data removeObjectAtIndex:sourceIndexPath.row];
    [self.data insertObject:fromCity atIndex:destinationIndexPath.row];
}
```

Not too fancy here, we just replace two objects given their indices. 

That's it! This is the typical process you would follow to use UITableView's built in editing feature. There's a lot more you can add on top of this such as animations and stylizing. Enjoy!

Cheers, 




