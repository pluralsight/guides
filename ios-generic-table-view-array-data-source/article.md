This guide is based on the post in [objc.io issue][objc-issue] but using the concepts describe there to make an implementation of a Generic Table View Data Source in Swift.

<h1>Previous solution in Objective-C</h1>

The previous solution, implemented in Objective-C, separate the array data source of the table view in a new class an uses blocks to configure each cell. We can see the *.h and *.m source files respectively:

<h2>ArrayDataSource.h</h2>

```objc

#import <Foundation/Foundation.h>


typedef void (^TableViewCellConfigureBlock)(id cell, id item);


@interface ArrayDataSource : NSObject <UITableViewDataSource>

- (id)initWithItems:(NSArray *)anItems
     cellIdentifier:(NSString *)aCellIdentifier
 configureCellBlock:(TableViewCellConfigureBlock)aConfigureCellBlock;

- (id)itemAtIndexPath:(NSIndexPath *)indexPath;

@end
```

<h2>ArrayDataSource.m</h2>

```objc

#import "ArrayDataSource.h"


@interface ArrayDataSource ()

@property (nonatomic, strong) NSArray *items;
@property (nonatomic, copy) NSString *cellIdentifier;
@property (nonatomic, copy) TableViewCellConfigureBlock configureCellBlock;

@end


@implementation ArrayDataSource

- (id)init
{
    return nil;
}

- (id)initWithItems:(NSArray *)anItems
     cellIdentifier:(NSString *)aCellIdentifier
 configureCellBlock:(TableViewCellConfigureBlock)aConfigureCellBlock
{
    self = [super init];
    if (self) {
        self.items = anItems;
        self.cellIdentifier = aCellIdentifier;
        self.configureCellBlock = [aConfigureCellBlock copy];
    }
    return self;
}

- (id)itemAtIndexPath:(NSIndexPath *)indexPath
{
    return self.items[(NSUInteger) indexPath.row];
}


#pragma mark UITableViewDataSource

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return self.items.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:self.cellIdentifier
                                                            forIndexPath:indexPath];
    id item = [self itemAtIndexPath:indexPath];
    self.configureCellBlock(cell, item);
    return cell;
}

@end
```

<h1>New solution in Swift</h1>
In the Swift implementation, we use a intermediate object to configure the data source, called ArrayDataSourceConfigurator. This class allow to the data source to don't implement some methods that don't belong in nature to it.

```swift

import UIKit

class IDArrayDataSourceConfigurator<T,U: UITableViewCell>
{
    var cellIdentifier: String
    var cellStyle: UITableViewCellStyle
    var configureCellClosure: ((cell: U, object: T) -> ())?
    
    private var items: [[T]]
    
    init(builder: IDArrayDataSourceConfigureBuilder<T,U>)
    {
        cellIdentifier = builder.cellIdentifier ?? ""
        cellStyle = builder.cellStyle ?? .Default
        configureCellClosure = builder.configureCellClosure
        items = builder.items ?? [[]]
    }
    
    func numberOfSections() -> Int
    {
        return items.count
    }
    
    func numberOfRowsInSection(section: Int) -> Int
    {
        return items[section].count
    }
    
    func item(indexPath indexPath: NSIndexPath) -> T?
    {
        return items[indexPath.section][indexPath.row]
    }
    
    func update(items items: [[T]])
    {
        self.items = items
    }
    
    func insert(item: T, indexPath:NSIndexPath)
    {
        self.items[indexPath.section][indexPath.row] = item
    }
}
```

Here we use to generics: **T** for the items used to configure the cell and **U** for the table view cell. 

[objc-issue]: https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/

