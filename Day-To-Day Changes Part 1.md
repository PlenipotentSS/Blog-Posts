#Biggest Day-To-Day Changes from Obj-C to Swift Part 1

Since Apple has released its new programming Language, Swift, developers around the world have been jumping in and seeing what is different! Yes, there are quite a bit of differences, such as Optionals, Generics, Extensions, Mutators, even custom Operators (the list goes on)... But for the every day Objective-C programmer, he/she has to know what they need to do get Swift up and running. To do that, let's look at a few examples that you would run into developing apps in Swift. The first part of this topic is focused primarily on syntax and variable declaration. Where the second part is the work flow of structures and classes.

## 1. Type Declaration & No Semi-Colons!

In Objective-C we begin each declaration with a specific type e.g. ```CGpoint``` or ```UIView```. If we didn't know the type, or didn't care we referred to it as ```id``` or ```instancetype```. While type is important in Swift, it can often be easily inferred what type the variable or constant should be. You can of course declare these types (e.g. Swift's String, Dictionary or even AnyObject), but you will be redundant in common cases. As you examine the code below also note that Swift does not use semi-colons to end the line of code. Let's jump in:

**Objective-C:**
```
//ex1
CGPoint aPoint = CGPointMake(9.0, 10.0);

//ex2
UIView *aView = [[UIView alloc] init];
```

In Swift, we have a smarter declaration, that can infer a type by what is on the right-hand-side. Now if we needed or wanted, we can always cast our variable to a specific type, but to do so it must go after the variable name:

**Swift:**
```
//ex1
let aPoint = CGPointMake(9.0, 10.0)
//or equivalently:
let aPoint: CGPoint = CGPointMake(9.0, 10.0)

//ex2
var aView = UIView()
//or equivalently:
let aView: UIView = UIView()
```

Now under the hood, Swift infers the specific type from its declaration. For this reason, when you are declaring any object you must declare it, otherwise you get the error *"Type Annotation Missing in Pattern."* Now there are ways to get around this, by using Optionals, but the beauty of Swift lies in its safety of knowing that a given variable or constant will have a value. Optionals are special structures that give a nil option as a value, and it promises 2 results. For example, say we are attempting to unwrap ```var someString:String?```, it will return:
	
- Nil, because there is nothing in it, OR
- The string value

For more information on Optionals, read *"The Basics of Optionals in Swift"*.

But wait... What happened to alloc] init]? Swift is built with efficiency in mind. It automatically keeps a reference count for you, and it also manages it so you don't even have to worry about allocating space! Lastly, you probably guessed why you don't see the "init"... ```()``` is init! so now all you have to do is call: ```myFavoriteFunction()``` without the hassle of memory management.

## 2. No More Brackets! Hello Dots!

That's right! The thing that Objective-C developers have come to hate and then love are no longer available. The move away from brackets syntax is now replaced with a somewhat more conventional syntax: the dot syntax. 

Now, even though dot syntax is used in Objective-C, Swift uses it almost exclusively for method calls and attributes.

**Objective-C:**
```
//ex1
CGFloat x = [aPoint x];
//or often used:
CGFloat x = aPoint.x;

[aView addSubView:[[UIView alloc] init]]];
```

In Objective-C, we can see the one common instance where dots are used (property retrieval). We also see how nesting brackets work to either retrieve attributes or make method calls on other objects. In Swift, we ignore the bracket syntax on object calls, and explicitly use dot syntax:

**Swift:**
```
//ex1
let x = aPoint.x	//equivalent to: let x:CGFloat = aPoint.x

//ex2
aView.addSubView(UIView())
```

Now from what we learned before, example 1 hasn't really changed much. We can still get the attributes of an object the same as Objective-C; we just have to know the declaration syntax. Now Example 2, is something different... functions are much closer to variables in Swift, you can move them around, pass them into functions - so it makes sense that you can now call a method as if it were a variable. Voila!



### 3. How to Declare: Constants (```let```) vs. Variables (```var```)
 
In Objective-C, we create our variables knowing that we will either add/remove information or ensure the variables were static/immutable. We did this because there were times where we wanted the data to be modified and times where we wanted it as static as possible. Creating these immutable objects ensured safety as well as better memory allocation. Let's take a look at Obective-C's Dictionary:

**Objective-C:**
```
NSDictionary *dict = @{@"Hello": @"World"};
```

This is a simple assignment of an ```NSDictionary``` to the variable ```dict``` with a single key-value pair: "Hello"->"World". ```dict``` is static and unchangeable (immutable). However, if we wanted to add or remove elements to a Dictionary object (mutable), we would have declared it like so:

**Objective-C:**
```
NSMutableDictionary *mutDict = [[NSMutableDictionary alloc] initWithDictionary:@{@"Hello": @"World"}];
[mutDict setObject: @"Hello Again" forKey: @"someKey"];	//add key-value
NSString str = [mutDict objectForKey:@"someKey"]; //retrieve value
```

Swift takes the idea of the immutable concept and made it available to all constants. Swift also has native data structures that you will want to use (Dictionary, String, Int, Double, and Array). You can still use the NSDictionary and NSMutableDictionary (and it is unclear whether Apple wants to move away from those structures), but the use of Swift's Dictionary is optimized and should therefore be used in our context.

The ```let ``` declaration defines a constant of any type, but does so in the thought that you can access it, compare it and even change it in certain circumstances - all while keeping an efficient use of memory.  *Let* us take a look:

**Swift:**
```
let aDictionary = ["SomeKey", "HelloWorld"]
```
This creates a constant that has a dictionary with a key-value pair: "SomeKey"->"HelloWorld". This is very similar to the notion of immutability, such that we cannot add or change the value of this given dictionary. Under the hood, when we create a constant dictionary, we are essentially doing this:

**Swift:**
```
let aDictionary: Dictionary<String, String> = ["SomeKey", "HelloWorld"]
```
Didn't expect that did you? Another thing about swift data structures is that you have to be explicity what you want stored in the dictionary: ```Dictionary<KEY : HASHABLE, VALUE>```. KEY is the type you will be using to set as the key, and the one requirement of this type is that it is hashable so that it can be referenced easily. VALUE is the value type you want your object to be stored as. The example below uses String at the KEY and AnyObject is the type for VALUE:

**Swift:**
```
var aMutDictionary = Dictionary<String, AnyObject>()
aMutDictionary["aKey"] = SomeObject	//add key-value
let obj: AnyObject = aMutDictionary["aKey"]	//retrieve value
```

Now you may be asking, why would I use one over the other? In  many circumstances, you will want a variable, as you will want to add to it, change it, and even replace it with a new instance. All this can be done in the ```var```. 

However, if you are just storing a string to retrieve the information, or store a static-amount of values in an array, you should use ```let```. Swift will optimize your app the best it can if you use more constant variables. ```let``` is also very helpful for when you want to store a value, ```let num = 5```. ```num``` can be accessed and even moved with the safety of knowing it will not change. Other reasons why ```let``` is very important to use is because the compiler will inform you as you are typing that there will be something wrong. You don't have to wait until runtime for your errors to be found, making the app process even easier!

***
###Next Up:
Stay tuned for Part 2 where we will cover Functions, Generics, and more. For now, a little teaser... One example that you will find different from Objective-C is that a function with Generics can take any variable as an input. One topic we will discuss next time is conforming to these generic functions. For the mean time, Generic Functions allow you to extend to purpose of one type to many many types (example: Counting Number of Elements):

**Swift:**
```
let str = "Hello World!"
let strLength = countElements(str)

// also:
let arr = ["Hello", "World!"]
let arrLength = countElements(arr)
```


***

```CountElements<T>()``` as it is written, uses generics to take any object T and returns its number of elements. As you can imagine there would be instances where this would go wrong, but if you are familiar with generics, we just have to find ways for our objects to conform to the method! Till next time!

