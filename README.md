# LLModel: Object Property Mapper for JSON

LLModel is a library for mapping the JSON data to object properties.

LLModel:
* supports recursive model initializations (see examples below)
* supports primitive values such as bool, char, integer, float and others
* handles NULL values gracefully
* handles Date and URL values properly

## Example usage

Say you have the following three models:
* User
* Address
* Feed

And the JSON data we will be receiving is going to be User which contains one Address and many Feeds.

Here I will show only the User model (download the example to see other models), you have to inherit from LLModel.

````
@interface User : LLModel
@property (strong, nonatomic) NSNumber *userId;
@property (strong, nonatomic) NSString *firstName;
@property (strong, nonatomic) NSString *lastName;
@property (strong, nonatomic) NSString *username;
@property (nonatomic) BOOL publicProfile;
@property (nonatomic) NSInteger loginCount;
@property (strong, nonatomic) NSDate *createdAt;
@property (strong, nonatomic) Address *address;
@property (strong, nonatomic) NSMutableArray *feeds;
@end
````

In the implementation file, you need to override the **initWithJSON:(id)JSON** method and define your mapping.
Mapping is very basic,
* The left side (dictionary keys) is your property names.
* The right side (dictionary values) is the JSON keys.

There is no need to worry about the types, LLModel will check the property types at runtime and assign the values accordingly.

````
@implementation User

- (id)initWithJSON:(id)JSON
{
    self = [super initWithJSON:JSON];
    if(self) {

        self.mappingDateFormatter = [[NSDateFormatter alloc] init];
        [self.mappingDateFormatter setDateFormat:@"dd.MM.yyyy"];
        
        NSDictionary *mapping = @{@"userId":@"id",
                                  @"firstName":@"first_name",
                                  @"lastName":@"last_name",
                                  @"username":@"username",
                                  @"publicProfile":@"public_profile",
                                  @"loginCount":@"login_count",
                                  @"createdAt":@"created_at"
                                  @"address":@"address",
                                  @"feeds":@{@"key":@"feeds",@"type":@"Feed"}};
        
        [self setValuesWithMapping:mapping andJSON:JSON];
        
        // check if there are any errors
        if(self.mappingErrors.count > 0) {
            [self logAllMappingErrors];
        }
    }
    return self;
}

@end

````

## Recursive Mapping

In the above example the most interesting part is the mapping of the "address" and "feeds" properties.

LLModel simply gets the values from JSON value and creates an Address object (Address model should also be a subclass of LLModel) and assigns to property.

Feeds is a little bit different story. Since **feeds** property is defined as NSMutableArray (or MSArray) LLModel can't know which type of objects it must contain.

So in the mapping, we provide a dictionary with the keys: "key" and "type". LLModel will than create the Feed objects and add them to NSarray and finally will assign to the property.

## Error handling

LLModel will add NSerror objects to self.mappingErrors if there are any errors while parsing the JSON. You can log the errors with the helper method [self logAllMappingErrors].

## Best Practice

I reccomend creating a BaseModel which will be a subclass of LLModel. All of your models then should subclass BaseModel instead of LLModel directly.
One of the main reasons for this is because many of the dates you receive from JSON will have the same date format, so in BaseModel you can define the date formatter only once.

## Example Project

Please download the example project to see how the enitre system works.


