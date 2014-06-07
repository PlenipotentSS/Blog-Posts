Morse Torch Part 1
-------

Ever wanted to make yet another morse code app? Here's the beginning of a three part blog post on how to use NSOperations and AVFoundation's Torch (iPhone's flashlight). First thing is prepare the typical interface, with a UITextField and a UIButton. We will be using user input to then convert to Morse Code Symbols/Words to send. Best way is put them into Storyboard and then make IBOutlet for the UITextField and a UIAction for the button. For my reference throughout the project, I'll be using the following properties in my transmit controller: 

```
@interface InputViewController () <UITextFieldDelegate>
@property (weak, nonatomic) IBOutlet UITextField *inputField;
@property (weak, nonatomic) IBOutlet UIButton *transmitButton;
@property (weak, nonatomic) IBOutlet UILabel *morseText;
@property (weak, nonatomic) IBOutlet UILabel *letterText;
@property (nonatomic) NSOperationQueue *torchQueue;
@property (nonatomic) SSProgressViewRingGradient *hudProgress;

@end
```

The UITextFieldDelegate I use only to disable the transmit button as the inputField is selected and vice versa. A cool add-on with anything that uses the UITextField delegate is the following method from UIResponder:

```
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    if ( [self.inputField isFirstResponder]) {
        [self.inputField resignFirstResponder];
    }
}
```
This method does one main thing. If the keyboard is active, then any touch action sent to the UI (sauf the keyboard) makes the keyboard disappear. This is particularly handy for UX for those who want the keyboard to disappear by just tapping outside the keyboard.

Letter to Symbol Conversion With a Dash of Validation
------------

For this part we are going to be creating a NSString category that will transfer a string into an array of morse code characters. This function I'll call +(NSString*) getSymbolFromLetter:. We use a class method (i.e. +) because we will not require an instance of the class to be instantiated for the method to create the string. Another viable way to do this is to instantiate an object with a string and then pass a local method to return an array with the given string. Typically you'll find class methods more in categories, so we'll continue that for our purpose. 

Walking through the methodology, we need the following to convert strings into symbols:

* 1. create a set or dictionary to map a character (letter) to a symbol (morse code for that letter).
* 2. validate the string to remove any non A-Z and 0-9 characters.
* 3. return the symbol for each valid letter.

I will not provide code to produce the dictionary/set, but it is easy to find any morse code conversion searching your favorite search browser. Therefore, we move to step 2. Validating the string is essentially qualifying the string to meet the requirements of the dictionary/set, therefore we can assume we want any character that we have a 1:1 map. I particularly like using Regular Expressions as it is a fast string replace given occurrences with "regex". Let's take a look:

```
+(NSString*) validateString:(NSString*) string {
    string = [string uppercaseString];
    
    NSError *error;
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"[^A-Z0-9]" options:NSRegularExpressionCaseInsensitive error:&error];
    string = [regex stringByReplacingMatchesInString:string options:0 range:NSMakeRange(0, [string length]) withTemplate:@""];
    if (!error) {
        return string;
    }
    return @"";
}
```

For a short lesson in regex, we take the regular expression patter "[^A-Z0-9]" and explain a little further. First the brackets result in any character (including repetitions) provided will be selected when searched. Here we mention that A-Z and 0-9 will be set to be selected. Finally the upward's carrot inverts the items selected resulting in a regular expression that searches for all NON A-Z and 0-9 characters. With that we then replace any of those non-capital-alpha-numeric letters with a empty string.

Next we have the actual returning of the symbol for each letter. First let's look at the public method for getting the symbol for the letters in a string, followed by the actual return from the dictionary/set.

```
+(NSArray*) getSymbolsFromString:(NSString *)string {
    string = [self validateString:string];
    NSMutableArray *symbols = [NSMutableArray new];
    for (NSUInteger i =0;i<[string length];i++) {
        [symbols addObject:[self symbolForLetter:[string substringWithRange:NSMakeRange(i,1)]]];
    }
    return [NSArray arrayWithArray:symbols];
}

/*** Symbol/Letter Retrieval ***/
+(NSString*) symbolForLetter: (NSString*) letter {
    return [[self symbolLetterDictionary] objectForKey:letter];
}
```

This is a fairly straight forward implementation. Now with this, we can add the category we created for NSString into our transmit controller and use the class methods on NSString. Let's take a last look at how that is done:

```
// above interface:
#import "NSString+MorseCode.h"
...
//implementing the category:
NSArray* morse = [NSString getSymbolsFromString:inputText];
```

And that's part 1! We have successfully converting our string into morse code words that we will use next time to transmit over the Torch mechanism! Stay tuned!

Cheers,
Steven