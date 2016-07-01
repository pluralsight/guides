Over the last 2 years I have been working in a program to make it possible to work with Data analytics by just knowing the basics.

# Introduction

Red Sqirl is a web-based big data application that simplifies the analysis of large data sets. With Red Sqirl, you can quickly access the power of the Hadoop eco-system, enhancing the productivity of data scientists and analysts. Red Sqirl analyzes massive amounts of data rapidly and cost-effectively. An open platform that users can extend, which simplifies the Hadoop ecosystem (Hadoop, Hive, Pig, HBase, Oozie, etc), so you don't have to master each of those underlying technologies.
Red Sqirl accesses your data through third-party processing and storage engines and organises them into packages. For example, the Red Sqirl Pig package gives you access to Apache Pig. Red Sqirl also classifies a reusable piece of analyses into models. 
Red Sqirl is Open source and Free.

You can find more in www.redsqirl.com
Video version of this tutorial: https://youtu.be/LL6adYq4YL4 


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0626d3ba-21d9-4a25-bc9c-035ef81a5b47.png)


## Let's get started 
- The idea here is to give a first overall view of Red Sqirl and what it can do.

###### Summary

<b>Step 1:</b> Learn about the Red Sqirl interface, </br>
<b>Step 2:</b> Learn how to copy data into HDFS, </br>
<b>Step 3:</b> Learn how to build a workflow, </br>
<b>Step 4:</b> Learn how to run that workflow </br>
<b>Step 5:</b> and finally how to extract the result from that workflow. </br>


Red Sqirl is a platform, that you can install directly on top of a Hadoop Cluster. This tutorial is based on the Red Sqirl Docker image.  https://hub.docker.com/r/redsqirl/cloudera/

At this point we’ll assume you already know what Red Sqirl is, and that you’ve installed it successfully.

If you want to follow along with this tutorial step by step on your local environment,  you’ll need to know
 where the Red Sqirl installation folder is; /opt/redsqirl-2.6.0-0.12/
 and have Red Sqirl Pig package already installed.




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b3af3151-52ef-4de9-b6a0-317545184119.png)


To Sign in, we’ll just use the OS username & password that was set up when installing Red Sqirl.
If you’re signing in for the first time, you’ll be prompted with a window about updating your footer menu. You can just click OK.




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a58c5aba-7f25-4da7-ac60-891d84745ed9.png)


###### Step 1. So this is our Red Sqirl interface it’s made up of 3 main sections.

So let’s outline the 3 main sections of Red Sqirl.

<b>Section 1. On the left we have two tabs:</b>

- The flow-chart canvas, this is where the workflow will be executed.
- The remote file system  - where we can connect to any ssh server.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd32b607-2dfa-41e2-916d-54eada73d760.png)



<b>The flow chart canvas: </b>

The flow chart canvas is where we create a data analysis workflow, and the actions that we can use for our analyses, we’ll find it at the bottom, in the canvas footer. 



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/732c9615-1039-477b-8382-8f5ef8f69018.png)


The remote file system:
This is where we can connect to any ssh server, for example you can ssh localhost, which will correspond to the server that Red Sqirl is installed on.

So if we click on the remote file system tab and then click on the plus button.	
Then just fill the form as necessary. 

And now we can see our remote file system. 




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c764c58f-1acf-4c63-bf9b-32fd4c628c22.png)


###### Section 2. On the top right is the help tab.

The help tab is made up of
- a main section where we can find the generic options
-  And an action section
    - We can find the action section by clicking this Home button,
    -  This section describes each of the drag & drop actions that we can use.




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/906cb99d-9526-4843-8a58-f1ef94a7d5ec.png)


###### Section 3. On the bottom right, we have the Hadoop Distributed File System





![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d13c1f36-c5d5-45a4-9901-14461a56af33.png)



So those are the three main parts of the Red Sqirl interface.

###### Step 2. Now, our next step is to get some data for us to analyse. 

Inside the Red Sqirl installation folder we’ve already given you some data for this tutorial.
So what we need to do now, is copy this data into our HDFS.



So to do this,
- We’ll first go back  to the Remote File System,
- We need to go into the Red Sqirl installation folder, in my case it is /opt/redsqirl-2.4.0-0.12
- We now just find the folder called “tutorial_data” and drag it over and drop it anywhere into our HDFS window.
- There are a few different files in this folder. For this tutorial we’ll be using the file named getting_started.txt
- Now we can click on tutorial_data and click on the getting_started.txt file, 


###### Step 3. Ok, so now that we have data in our HDFS, we can start an analyses! 

To do this we just have to go back to the flowchart canvas section. 

So, now for our analysis, what we’re going to do is:
1. Configure the source for a hdfs file
2. Select the sum of two columns
3. Get the average of a column
4. Get the sum of a column when grouping by another
5. Join the two tables
6. Filter a table with a condition

To do all of this we first need to change the footer.
The action footer is the little frame on the bottom left of the FlowChart tab.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/778b6f58-50e4-41b8-90f1-a60f40daab06.png)


1. Click the green information symbol.
2. Once the configuration popup has appeared in the left column click “+”.
3. Type “extraPig” on the new empty line.
4. Click the “...” symbol.
5. On the new window, select redsqirl_pig in the drop-down menu
6. Click on the check box next to pig_audit, and click on the Select button.
7. Click OK.
8. Click OK.

You should see a new footer tab called “extraPig” with “Pig Audit” inside. To remove this new menu, you need to do the following.

1. Click the green information symbol.
2. Click on the check box beside “extraPig”.
3. Click on the Delete button (bin icon) in the table header.
4. Click OK.



<b>So let’s start to analyse the data. </b>
  


1. An analysis always starts with a source action, so let’s set this up first.
2. In the actions footer drag a new source icon onto the canvas.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8ac59bb6-73b3-4071-962a-2e5daf8165d3.png)


1. Now we just double click on it to configure it
2. Name the action “communication“.
3. Comment the action “Configure tutorial data“.
4. Click OK.
5. Now we’ll see a step-by-step window. 
6. Select “Hadoop Distributed File System“ as the data type then click next.
7. Select “TEXT MAP-REDUCE DIRECTORY” as the data subtype and click next.
8. Next we have to chose the path, that we saved our file into.
9. Click on the search button, and click the refresh button, then find the tutorial data file and then the getting started file pig_tutorial_data.mrtxt
10. Click on the radio button beside “pig_tutorial_data.mrtxt”- if you cannot find it, refresh the view by clicking on the search button- and click OK.
11. And now our data is in the source action. At this stage, you will see the data correctly displayed on the screen, the name of the fields are “Field1 Long, Field2 Long...”
12. In this next step we can change the field name and their type by clicking the edit button on the top-left of the table
- Here we’ll rename the fields.
- Copy and paste “subscriber_number STRING , Friend STRING , offpeak_voice INT , offpeak_sms INT , offpeak_mms INT ,peak_voice INT,peak_sms INT , peak_mms INT , sna_weight INT , subscriber_onnet INT ,friend_onnet INT” into the value field
1. Click OK. You will have the confirmation that the Header is correct.
2. Click OK to exit from the Configuration window.
3. If you leave the mouse cursor on the source action you will be able to see some configuration details
4. Save the Workflow by going into File > Save, name it “pig_tutorial”. By default it is saved in redsqirl-save HDFS directory and the file will have the extension ‘.rs’. Click OK to save.



<b>Now we can see the data with the new field names we just changed.</b>



And that’s it, we’ve set up the source for the data.

We can now see that the arcs around the source action icon have changed. 

The arcs around the icons, give information about the status of that action. 

To check what the arcs mean,  just click on the legend on the top left of the canvas, to hide it, just click it again.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3674eec8-1af8-488b-9057-2b791dfc9754.png)



So let’s re-cap:
- We’ve gone through the interface
- We’ve copied our data into HDFS
- And we’ve set up our source file and configured it

Now we’re ready to start the data processing.

Set Up a Pig Aggregator Action:

Pig Aggregator is an action in which aggregation methods are allowed to be used when selecting columns as you would in an sql statement. These aggregation methods are AVG , MAX , SUM etc. This action will group by either the selected attributes or all (default if none is selected).

On the Pig footer menu, we can select the aggregator action. 

1. Drag a pig aggregator action to the canvas.
2. Create a link between the source that was just configured and the new pig aggregator action by clicking between the image and the arc of the source action and then clicking on the pig aggregator image. (you’ll see an arrow connecting the two icons)
3. Open the new pig aggregator icon, name the element “nl_sum” and Click OK.
4. Now this first page, lets us choose the field that we want to aggregate by. For this we want to aggregate by “subscriber_number” Select “subscriber_number” and click "Select" in the Group by interaction.
5. Click next.
6. On this second page we can do the operation field by field. 
7. Select the copy from the dropdown menu on the generator interaction and click OK. We can also have a more granular choice of generation by clicking on the “Configure” button.
- Each Tab is a different configuration option
8. We can also sort the rows - On the top of the table click the “+” symbol to add a new row to the table.

One thing to note:  the check box on each row is only used for sorting and deleting, we don’t need to have each row ticked in order to continue

1. Click on the pen in Operation field of the new row and click the “SUM()” function and add the parameters “communication.offpeak_voice” and “communication.peak_voice”. In between the parameters add a “+” symbol so that the operation would read “SUM(communication.offpeak_voice + communication.peak_voice)”
2. click OK.
3. In the Field Name, type “total_voice” for the new column and change the type to DOUBLE.
4. Click next.
5. The next page shows that we can sort the data, we won't sort it now so click next.
6. The following page is about filter and format, click OK to leave the default parameters.
- The filtering option is another text editor where we can just create a condition. 
- We can choose the delimiter of the output, let’s use comma in the delimiter box 
- By default, the output type is compressed. In Red Sqirl the output format is important for linking one action to the next, the two types need to be compatible.
If we want we can also choose to do an audit of the output. 

1. Press ok;


###### Step 4. Ok so now let’s run the workflow.

Go on menu click Project and click on “Save & Run” 

Now since the Pig Action is using Map-Reduce this’ll take a minute or so to run.
- While this is running, the canvas can’t be modified 
- But what we can do, is create more workflows while it’s running, click on a new tab and drag and drop something onto a new canvas
- We can create and run multiple workflows at the same time.
- Also while the workflow is running we can check it’s progress in the Process Manager on the right hand side 



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/24f93041-978b-42c2-9e88-8c7c6e4a6ef9.png)


- click on the Process Manager tab and hit refresh
- In here we’ll see all of our processes 
- and if we want, we can pause or kill any of them
- We can also check the logs in the Oozie Control - click on the left green button 


And that’s it! 
As soon as the process is finished, we can see the results.
- We can just go to the Pig Aggregator action and go into options and click on “data output” 
- And here we can see the results 
- We can see the path, where we can find it at the top, and we can download it as a CSV 





###### Pig Aggregator Action

1. Drop another pig aggregator onto the canvas.
2. Make a link between the source and the new pig aggregator.
3. Open it, and give it the name "comm_groupbyall" and click OK.
4. This time leave the group by list alone so nothing is selected and click next.
5. Create a new row (+ button).
6. In this new row copy and paste “AVG(communication.offpeak_voice + communication.peak_voice)”.
7. Call the field “total_voice_avg” and select the DOUBLE type.
8. Click next.
9. Click next on the sorting page.
10. Click OK on the final page.

###### Perform a Pig Join Action

<b>To make each dataset interactable with each other it is necessary to perform a join on them.</b>

1. Drop a pig join onto the canvas.
2. Create a link from “comm_groupbyall” to the new pig join action.
3. Create a link from “nl_sum” to the new pig join action.
4. Double click the pig join and call it “nl_vs_total”.
5. The first page list the table aliases, click next.
6. On the following page, make sure that “copy” is selected as the generator and click OK.
7. Click next.
8. This page has two interactions that specify the join type and the fields to join on, we use the default join type which is “Join” so this does not need to be changed.
9. In “Join Field” column, type “1” in the two rows. This condition will join the two tables together.
10. Click next.
11. Click next on the sorting page.
12. Click OK on the final page.


###### Step 5. Filter a Data set

Now we want to make a condition to see what subscribers have a higher total voice calls than the average of the entire dataset. The easiest would be to add the condition in Join but we will create a new select for demonstration purposes.

1. Drop a new pig select action onto the canvas.
2. Create a link from the pig join action to the new pig select action
3. Open it and change the element id to be “high_voice”.
4. In the generation drop-down menu, select copy and click OK.
5. Click next.
6. In the Sorting page, select “nl_sum_total_voice” and “DESCENDING” order.
7. Click Next.
8. Click on the Pen in the Condition section and write “nl_sum_total_voice > comm_groupbyall_total_voice_av”.
9. Click OK and then finish the action by clicking OK.

We would now like to keep “nl_vs_total” intermediate result before running the workflow.

1. Leave your mouse on “nl_vs_total”. Click Options button and select “Data Output” from the drop-down menu.
2. Select “BUFFERED” instead of “TEMPORARY”.
3. Click OK.

You can now “Save and Run”, and see your result in the Data Output of “nl_vs_total” and “high_voice”
To see the results, leave your mouse on the action “nl_vs_total” or “high_voice”, > Options > Data Output.
You can close it by hitting “Cancel” or “OK”
Once you are happy with the result you can clean all the data generated by this workflow by clicking on “Select All” and then “Clean Actions” in the “Edit” top menu.




Final canvas should be something like this;



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a890dda0-a82c-498c-8896-f1d72774f263.png)




# Conclusion

Red Sqirl is a program that makes it possible to create and use data analyses ideas in an easy way. Fast to create, fast to share and easy to reuse.

- We’ve covered the interface
- We copied data into HDFS
- We built a workflow
- And then we extracted the results from that workflow
- We configured the source for a hdfs file
- Selected the sum of two columns
- Got  the average of a column
- Got the sum of a column when grouping by another
- We joined the two tables
- And filtered a table with a condition


