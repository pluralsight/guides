This tutorial will cover using MySQL to perform operations on data tables containing transaction information. Our primary example will involve money transactions.

# Transactions and table data

Consider a digital transaction in which a value is stored in one place and the context is stored elsewhere. For example, when a cash transaction occurs, we can represent the direction of the cash transaction in a table and the amounts in another table to dodge negative numbers. Similarly, there are a lot of forums and online commenting systems in which you can upvote or downvote. These votes will be stored separate from their individual count, thereby having entirely positive values, but they will be linked via foreign key to a table containing the original values of each vote, whether they are positive or negative.

## Money transaction example

I encountered a similar problem related to different money transactions. The type of transaction would decide whether it was an incoming transaction or an outgoing one. Modeling this same circumstance, we are going to look at three things today:

* Connecting the transactions by their value.
* Creating a multiplier based on the relation for the value.
* Creating a consolidated sum grouped on different currencies, and users.

# Grouping

When grouping, **aggregate functions** are key. We will be applying certain aggregate functions on a sample table containing financial details. The table's Data Definition Language (DDL) is provided in this post. 

The structure of the table looks like:

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

For this table, I had added a new column that tells the direction. `1` being credit and `2` being debit. The values `1` and `2` do not mean anything to us, but they serve as a foreign key to another table that holds the values. I have intentionally used positive numbers to mimic the real-world scenario.

Consider the sample data set shown below, which represents values for one user:

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

# Trials

## Getting the total amount.

To get the sum of all the amounts, use the `SUM` and `GROUP BY` functions. In the above case, we'll use the following query to get the sum of all the transactions, irrespective of the user, direction and/or the currency.

    SELECT SUM(`Amount`) AS `Total` FROM `transactions`;

The above will output something like:

    ╔═════════╗
    ║  Total  ║
    ╠═════════╣
    ║ 4185.00 ║
    ╚═════════╝

## Credit and Debit

This is not our ideal case. We have different rows getting different values. On top of all that, the amount debited should be negated from the value. In that case, we can split the income and expenditure using the `GROUP BY` function, which gives us the following query:

    SELECT SUM(`Amount`) AS `Total` FROM `transactions` GROUP BY `Direction`;

The above query will yield us something of the form:

    ╔═════════╗
    ║  Total  ║
    ╠═════════╣
    ║ 2150.00 ║
    ║ 2130.00 ║
    ╚═════════╝

This table signifies a list of the sums of all transactions going in the same direction.

## Different Currencies

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

As seen above, this command creates an additional split in the data. By adding the `Currency` category, we group first by direction and next by currency, giving us four different values.

However, our output is rather confusing because we don't know what each number signifies. Let's add more columns to reveal what the rows mean:

    SELECT SUM(`Amount`) AS `Total`, `Currency`, `Direction` FROM `transactions` GROUP BY `Direction`, `Currency`;

The query now looks better:

    ╔═════════╦══════════╦═══════════╗
    ║  Total  ║ Currency ║ Direction ║
    ╠═════════╬══════════╬═══════════╣
    ║  150.00 ║ GBP      ║         1 ║
    ║ 2000.00 ║ USD      ║         1 ║
    ║  135.00 ║ GBP      ║         2 ║
    ║ 1995.00 ║ USD      ║         2 ║
    ╚═════════╩══════════╩═══════════╝

Comparing to the initial data set, can you tell what effect the `GROUP BY` command had on our query? The values appear in the order dictated by the `GROUP BY` scheme. The output is sorted by `Direction` first and then by `Currency`, as seen by the fact that `GBP` comes before `USD` in the alphabet.

Can you see the effect of using `AS` in the query? Clearly, before we used this keyword, the query output kept the same order and values. However, we could not see the other information, such as the columns that you see now. Therefore, `AS` adds more information to the returned table in the form of output columns.

## Multiplier Query

> "Direction?" No idea.

But still, we have literally no clue what the direction means to us. Also, since we know `1` is positive (stays the same `1`) and `2` is negative (should be converted into `-1`), we need to change them to their corresponding values. To make this kind of change, we use *SQL JOINs* if they are using the foreign key table relation or we use `CASE … WHEN … END` statements if we are only dealing with values. 

Since our sample table only has values, we can build our multiplier.

    CASE
        WHEN `Direction`=1 THEN  1
        WHEN `Direction`=2 THEN -1
    END

The above gives you the converted version of the `Direction` column, or the values. So having this code with the original `Direction` like this:

    SELECT `Direction`, (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    ) AS `Multiplier`
    FROM `transactions`

The resulting query makes real sense:

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

## Combining Both

Looks like we have got what we needed: the multiplier! When the value of the result is multiplied with the amount, the total amount can be summed. With the multiplier as well as a parameter, the SQL query looks like:

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

## Combine and Compress

With the currency, direction and multiplier, we have gotten to a point where we can understand what is happening, at least to an extent. So, let us add another field that shows the right total amount with the sign. We just need to multiply the total with the multiplier.

> **Pro Tip:** If you try using the column `Multiplier`, it will not work. That column is for viewing purposes as an alias only. No column actually exists in that name. 

Instead, we should redefine the contents of the column in order to generate the new column. Let's say the new column is `Total` and get rid of the old `Total` column. In short, we are multiplying what was already in `Total` with whatever we get in the multiplier query. That way we get the values we want, and we can get rid of the `Direction` since we are not using it.

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
As you can see, the `Total` and `Currency` columns result from the `AS` keyword, and the sign comes from the `*`, meaning multiply, in the select statement.

## Getting net values

Now we can clearly decipher the different currencies and their values. With a small change in the query, we can combine them as a final sum, adjusted for the negative and positive interactions alike.

We can get rid of the `Direction` from grouping and change the parentheses, so that the `SUM` will take care of the whole thing including multiplication:

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

Here, we see the effect of using `GROUP BY`. 15.00 GBP comes from adding 150 to -135. Similarly, 5.00 USD comes from adding 2000 to -1995. 

This way, we can get the consolidated sum of different entities (`Currency`) based on their values (`Direction`). If we need to separate by the users as well, we just need to add another `GROUP BY` value of that column. So, that would be like:

    SELECT SUM((`Amount`) * (
        CASE
            WHEN `Direction`=1 THEN  1
            WHEN `Direction`=2 THEN -1
        END
    )) AS `Total`, `Currency` FROM `transactions` GROUP BY `Currency`, `UserID`;

And finally using the JOINs, you can get the complete details of each user, along with their corresponding currency balances. 

___________


I hope this article was interesting and helpful. Please leave all comments and feedback in the discussion section below. And, as always, don't forget to hit the favorite button! Thanks for reading.