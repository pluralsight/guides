There are a lot of instances where only the value is stored and the context is stored somewhere else. For example, when a cash transaction happens, the direction of the cash transaction will be noted in a separate table and there will not be any negative numbers. The same way, there's another case that could be possible. There are a lot of fora and online commenting systems where you can vote up or down. The votes will be stored elsewhere and they will be all positive integers and will be linked via foreign key to a different table, where the original values of each vote will be stored, either positive or negative.

I was working on a similar case, where it involves different money transactions, and the type of transaction will decide whether it is an incoming transaction or an outgoing one. We are going to look at three things today:

* Connecting the transactions by their value.
* Creating a multiplier based on the relation for the value.
* Creating a consolidated sum grouped on different currencies, and users.

## Grouping

The first thing that comes to our mind when grouping is the aggregate functions. We will be using the aggregate functions to apply on a sample table that has all the financial details. The table's DDL is provided in this post. The structure of the table looks like:

### Sample Table

    ╔═══════════╦═══════════════════════════════╗
    ║  Column   ║             Type              ║
    ╠═══════════╬═══════════════════════════════╣
    ║ TranID    ║ int(11) Auto Increment        ║
    ║ Subject   ║ varchar(100)                  ║
    ║ Amount    ║ decimal(7,2)                  ║
    ║ Currency  ║ char(3)                       ║
    ║ Direction ║ int(11)                       ║
    ║ UserID    ║ int(11)                       ║
    ║ TimeStamp ║ timestamp [CURRENT_TIMESTAMP] ║
    ╚═══════════╩═══════════════════════════════╝

For this table, I had added a new column that tells the direction. `1` being credit and `2` being debit. The values `1` and `2` don't mean anything to us, but technically, out in the real world, it's going to be a foreign key to another table that holds the values. I have intentionally used positive numbers to mimic the real-world scenario.

### Sample Data

A sample data of the contents of the table, which we currently use for one user is as below:

    ╔═════════╦═══════════════════════╦═════════╦══════════╦═══════════╦════════╦═════════════════════╗
    ║  TranID ║        Subject        ║ Amount  ║ Currency ║ Direction ║ UserID ║      TimeStamp      ║
    ╠═════════╬═══════════════════════╬═════════╬══════════╬═══════════╬════════╬═════════════════════╣
    ║       1 ║ Salary                ║ 2000.00 ║ USD      ║         1 ║      1 ║ 2017-05-12 09:15:56 ║
    ║       2 ║ Sold TV               ║  150.00 ║ GBP      ║         1 ║      1 ║ 2017-05-12 09:16:23 ║
    ║       3 ║ Mobile Bill           ║   25.00 ║ GBP      ║         2 ║      1 ║ 2017-05-12 09:19:03 ║
    ║       4 ║ House Rent            ║ 1500.00 ║ USD      ║         2 ║      1 ║ 2017-05-12 09:21:03 ║
    ║       5 ║ Food                  ║   10.00 ║ GBP      ║         2 ║      1 ║ 2017-05-12 09:24:33 ║
    ║       6 ║ Internet Bill         ║  250.00 ║ USD      ║         2 ║      1 ║ 2017-05-12 09:26:03 ║
    ║       7 ║ Value Added Tax       ║  100.00 ║ GBP      ║         2 ║      1 ║ 2017-05-12 09:26:45 ║
    ║       8 ║ Credit Card Bill      ║  150.00 ║ USD      ║         2 ║      1 ║ 2017-05-12 09:27:46 ║
    ║       9 ║ Import Charges        ║   75.00 ║ USD      ║         2 ║      1 ║ 2017-05-12 12:27:46 ║
    ║      10 ║ Cash Point Withdrawal ║   20.00 ║ USD      ║         2 ║      1 ║ 2017-05-12 12:28:20 ║
    ╚═════════╩═══════════════════════╩═════════╩══════════╩═══════════╩════════╩═════════════════════╝

## Trials

### Getting the Total amount.

The first thing to get the sum of all the amount is to use `SUM` and `GROUP BY` functions. In the above case, we'll use the following query to get the sum of all the transactions, irrespective of the user, direction or the currency.

    SELECT SUM(`Amount`) AS `Total` FROM `transactions`;

The above will output something like:

    ╔═════════╗
    ║  Total  ║
    ╠═════════╣
    ║ 4185.00 ║
    ╚═════════╝

### Credits and Debits

This is not our ideal case. We have different rows getting different values. On top of all that, the amount debited should be negated from the value. In that case, we can split the income and expenditure using the `GROUP BY` function, which gives us the following query:

    SELECT SUM(`Amount`) AS `Total` FROM `transactions` GROUP BY `Direction`;

The above query will yield us something of the form:

    ╔═════════╗
    ║  Total  ║
    ╠═════════╣
    ║ 2150.00 ║
    ║ 2130.00 ║
    ╚═════════╝

### Different Currencies

With this, in the same way, we will be able to add another grouping by giving the column name separated by a comma. So if we need to add the next stage of grouping, let's add the `Currency` column this way:

    SELECT SUM(`Amount`) AS `Total` FROM `transactions` GROUP BY `Direction`, `Currency`;

The above yields:

    ╔═════════╗
    ║  Total  ║
    ╠═════════╣
    ║  150.00 ║
    ║ 2000.00 ║
    ║  135.00 ║
    ║ 1995.00 ║
    ╚═════════╝

### Some clues please?

Since for a long time, we have literally no clue what's what. Let's add more columns and indicate the rows, and what they mean. Now our query contains more columns:

    SELECT SUM(`Amount`) AS `Total`, `Currency`, `Direction` FROM `transactions` GROUP BY `Direction`, `Currency`;

The query now gets somewhat better, showing the following:

    ╔═════════╦══════════╦═══════════╗
    ║  Total  ║ Currency ║ Direction ║
    ╠═════════╬══════════╬═══════════╣
    ║  150.00 ║ GBP      ║         1 ║
    ║ 2000.00 ║ USD      ║         1 ║
    ║  135.00 ║ GBP      ║         2 ║
    ║ 1995.00 ║ USD      ║         2 ║
    ╚═════════╩══════════╩═══════════╝

### Multiplier Query

> "Direction?" No Idea.

But still, we have literally no clue what the direction means to us. Also, since that we know, `1` is positive (stays the same `1`) and `2` is negative (should be converted into `-1`), we need to change them to their corresponding values. To make this kind of change, either we use *SQL JOINs*, if they are using the foreign key table relation or `CASE … WHEN … END` statements, if they are just values. Since our sample table has only values, let's build our multiplier.

    CASE
        WHEN `Direction`=1 THEN  1
        WHEN `Direction`=2 THEN -1
    END

The above gives you the converted version of `Direction` column, or the values. So having this code with the original `Direction` like this:

    SELECT `Direction`, (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    ) AS `Multiplier`
    FROM `transactions`

The above query gives us the following result, which makes real sense.

    ╔═══════════╦════════════╗
    ║ Direction ║ Multiplier ║
    ╠═══════════╬════════════╣
    ║         1 ║          1 ║
    ║         1 ║          1 ║
    ║         2 ║         -1 ║
    ║    * 03 more rows *    ║
    ║         2 ║         -1 ║
    ╚═══════════╩════════════╝

\* The above has some rows that are skipped.

### Combining Both

Looks like we have got what we needed. It's the multiplier! When the value of the result is multiplied with the amount, the total amount can be summed. With the multiplier as well as a parameter, the SQL query looks like:

    SELECT SUM(`Amount`) AS `Total`, `Currency`, `Direction`, (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    ) AS `Multiplier` FROM `transactions` GROUP BY `Direction`, `Currency`;

The above query gives us the following output:

    ╔═════════╦══════════╦═══════════╦════════════╗
    ║  Total  ║ Currency ║ Direction ║ Multiplier ║
    ╠═════════╬══════════╬═══════════╬════════════╣
    ║  150.00 ║ GBP      ║         1 ║          1 ║
    ║ 2000.00 ║ USD      ║         1 ║          1 ║
    ║  135.00 ║ GBP      ║         2 ║         -1 ║
    ║ 1995.00 ║ USD      ║         2 ║         -1 ║
    ╚═════════╩══════════╩═══════════╩════════════╝

### Let's Combine and Compress

With the currency, direction and multiplier, we have gotten to a point where we can understand what's happening, well, at least to an extent. So, let's add another field that shows the right total amount with the sign. We just need to multiply the total with the multiplier.

> **ProTip:** If you try using the column `Multiplier`, it will not work. It is just for viewing purposes as an alias only and there's really no column that exists in that name. We should redefine the contents of the column for generating the new column. Let's say the new column as `Total` and get rid of the old `Total` column. In short, we are multiplying what was already total with the query for multiplier. Also, we can get rid of the `Direction` since we are not using it.

The updated query now looks like:

    SELECT SUM(`Amount`) * (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    ) AS `Total`, `Currency` FROM `transactions` GROUP BY `Direction`, `Currency`;

The output for the above query looks like:

    ╔══════════╦══════════╗
    ║  Total   ║ Currency ║
    ╠══════════╬══════════╣
    ║   150.00 ║ GBP      ║
    ║  2000.00 ║ USD      ║
    ║  -135.00 ║ GBP      ║
    ║ -1995.00 ║ USD      ║
    ╚══════════╩══════════╝

### Grand Total

Now we can clearly decipher the different currencies and their values. With a small change in the query, we can combine them as a final sum. If we get rid of the `Direction` from grouping and change the parentheses, so that the `SUM` will take care of the whole thing including the multiplication this way:

    SELECT SUM((`Amount`) * (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    )) AS `Total`, `Currency` FROM `transactions` GROUP BY `Currency`;

We get the output like:

    ╔═══════╦══════════╗
    ║ Total ║ Currency ║
    ╠═══════╬══════════╣
    ║ 15.00 ║ GBP      ║
    ║  5.00 ║ USD      ║
    ╚═══════╩══════════╝

This way, we can get the consolidated sum of different entities (`Currency`) based on their values (`Direction`). If we need to separate by the users as well, we just need to add another `GROUP BY` value of that column. So, that would be like:

    SELECT SUM((`Amount`) * (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    )) AS `Total`, `Currency` FROM `transactions` GROUP BY `Currency`, `UserID`;

And finally using the JOINs, you can get the complete details of each user, along with their corresponding currency balances. Hope this article was interesting and helpful. Please do let me know in comments like if there's a better way to achieve a few things.