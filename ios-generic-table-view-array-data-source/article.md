This guide is based on the post in [objc.io issue][objc-issue] but it tries to explain how to expand the use of generic table view array data sources to separate the logic of filling the cells of a table view from the logic of the controller.

The main problem of the solution implemented in [objc.io issue][objc-issue] (apart than it used Objective-C) is that you can't reuse the array data source architecture in different table views so you have to implement the solution once per view controller. With the code implemented here, you'll have a reusable solution that can be used in all your table views. Otherwise, there are some problems that this solution can't afford and we'll see why and possible solutions at the end

<h1>Previous solution</h1>

The previous solution

```objc
@implementation ArrayDataSource

- (id)itemAtIndexPath:(NSIndexPath*)indexPath {
    return items[(NSUInteger)indexPath.row];
}

- (NSInteger)tableView:(UITableView*)tableView 
 numberOfRowsInSection:(NSInteger)section {
    return items.count;
}

- (UITableViewCell*)tableView:(UITableView*)tableView 
        cellForRowAtIndexPath:(NSIndexPath*)indexPath {
    id cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier
                                              forIndexPath:indexPath];
    id item = [self itemAtIndexPath:indexPath];
    configureCellBlock(cell,item);
    return cell;
}

@end
```



[objc-issue]: https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/

