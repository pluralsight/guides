### The Story

This is a very simple Alexa skill.  It is a quiz to test your knowledge of world's capitals and countries.  It suits children and adults alike.  It is a fun game for families and friends.

This game is very easy to play.  It will ask you a set of 5 questions about capitals and countries.  It will go like this:

>You: Alexa, play Capitals and Countries.

>Alexa: There is no game in progress. Do you want to start a new game?

>You: Yes

>Alexa: Capitals and Countries. I will ask you 5 questions, try to get as many right as you can. Just say the number of the answer. Let''s begin. Question 1. Bogota is the capital of which country? 1. Russia. 2. Canada. 3. Thailand. 4. Colombia.

>You: four

>Alexa: That answer is correct. Your score is 1. Question 2. London is the capital of which country? 1. Spain. 2. Turkey. 3. Cuba. 4. United Kingdom.

>You: four

>Alexa: That answer is correct. Your score is 2. Question 3. Baghdad is the capital of which country? 1. Iraq. 2. India. 3. Cuba. 4. Spain.

>You: three

>Alexa: That answer is wrong. The correct answer is 1: Iraq. Your score is 2. Question 4. Vienna is the capital of which country? 1. Thailand. 2. Austria. 3. United Kingdom. 4. Belgium.

>You: two

>Alexa: That answer is correct. Your score is 3. Question 5. Havana is the capital of which country? 1. Egypt. 2. Iraq. 3. Sweden. 4. Cuba.

>You: four

>Alexa: That answer is correct. You got 4 out of 5 questions correct. Thank you for playing!



### Things used in This Project

#### Software and services

Alexa Skill Kit

Amazon Web Services AWS Lambda



### How it was made

Assuming that you already have a developer account, follow theses steps to build this custom skill.


#### Code a AWS lambda function for the skill


1. Select Lambda from Compute service after login to [Console](https://aws.amazon.com/).  If you do not have an account, you can create an AWS account.

2. Skip "Select Blueprint"

3. In "Configure Function", enter "Name/Description/Runtime"

4. Select the ‘Code Entry Type’ as ‘Edit Code inline’ and copy/paste the Lambda function from code section below.  Remember to change the App Id.

5. Set the handler and role as follows: (a) keep handler as "index.handler" and (b) add a new role for "lambda_basic_execution"

6. You will be asked to setup your IAM role if you have not done so.

7. Keep the advanced setting as default.

8. Select "Next" and review.  Then "Create Your Function"

9. Next create an Event Source.  Select Add event source and Select type as Alexa Skills Set

10. Copy the ARN for your lambda function.  You will need it for setting up your skill.



#### Setup up skill in Developer Portal




#### Testing the Capitals and Countries skill













Or load the live markdown tutorial to check the syntax.

this is a test 

will add more later

