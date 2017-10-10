There are sequences in all things we do – logically speaking. If I was to build a house, I need to first lay the foundation, and then the walls – followed by the roof. I cannot do the roof first and then start with the foundation.

Likewise, in the software development world, you first gather the requirements, analyze it, design (HLD followed by LLD), come up with user stories, write the code, write the test cases, execute the tests, and then deploy. Moving the sequences of these activities is like building the roof before the foundation – which is exactly what we are set out to do in Test Driven Development.

<h1>What is Test Driven Development?</h1>

Test Driven Development (TDD henceforth) is an iterative process where the test cases are written and executed first, and then the coding is done. This practice is contrary to the tradition involving coding first-testing next.

Primarily in a TDD environment, unit test cases are written and executed, followed by the code development. I will talk about the process involved in the next section.

By writing the unit test case first, the developer is setting a target that is in sight. The unit test cases are meant to cover a small portion of the functionality. Once the target is set through the test cases, the developer will write just enough code to make the test case pass – not more than he needs to.

When the unit test passes, he then writes the next unit test case – creates a new target. This is followed by code – which takes him to the target. 

The premise is to write short unit test cases and make the entire cycle of testing and coding as small as possible. More the number of iterations, the realization of benefits coming out of TDD are effective.

In this iterative process, the test cases guide the developer in writing the code. Hence, the methodology is called as test driven development.

<h2>When did TDD Start?</h2>

This is a difficult question to answer. There is no definite answer.

Usually, TDD is coupled with agile project management as agile too seeks to work in iterations and believes in building in smaller bits. But, the principle of writing test cases first followed by the code dates back to 1950s. 

Kent Beck who is credited for creating Extreme Programming is a major backer of TDD methodology – hence the revival of TDD. He refers to TDD as rediscovery in the context of agile development. 

Beck writes that the principle of TDD is quite old – employed during the age of mainframes. The practice that was followed by developers was to write down the expected output and then write the programs on punch cards. This was especially done as the time they got to play with systems was limited, and this was the best possible way for efficient use of their time. The output achieved on mainframes were compared to the documented output, and this would tell them if their program worked or not.

In the rediscovery phase, TDD was re-introduced by the Smalltalk community on their SUnit framework. When Java came to the fore, JUnit framework was the tool employed for executing automated unit tests, and Beck was one of the developers of JUnit. Due to the commonalities between the unit test framework, agile revolution, and Beck, TDD was reintroduced into the mainstream yet again.

<h1>The TDD Process</h1>

At a high level, the process to develop a software using TDD is provided in figure 1.


<center>![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3aa56c9e-529b-40b3-b71b-2ced1a44b922.jpg)
<br /><br />
Figure 1: Process for Developing Software using TDD</center>

<h2>Step 1: Write Unit Test Case</h2>
As mentioned earlier in the post, the first activity that you do in TDD is to write a unit test case. To reiterate, the unit test case must be as small as possible, as long as it can be independently tested.

Let’s say that I am developing a login page for an application. The login page has a logo in the header section followed by fields for username and password. Finally, there is a submit button to input the credentials.

In step 1, I start with the unit test case for the logo that I am going to place on the web page. Nothing more nothing less.

<h2>Step 2: Execute Test</h2>
At this point in time, I have a unit test case and absolutely no code. Yet, I am going to execute the test case. Why? In TDD, <strong>you always write code for a failing test</strong>, and this is the reason why we execute the test case knowing fully well that there is no code to support it.

Executing the test case without code also provides the insurance that the developer does not get carried away by the need to have the code in place before writing the test case. It is an additional layer of control that ensures that TDD is followed true to the spirit of it.

<h2>Step 3: Write Code</h2>
After the test has failed, you then write just enough code to execute the test case successfully. You should not write more code than needed. 

It is a common practice among developers to write plenty of code, more like a novel rather than the bullet points that we are looking for in TDD.

<h2>Step 4: Execute Test</h2>
Now that we have the code that is needed to successfully pass the unit test case created in step 1, we execute the test again.
If the test fails, it indicates that the code developed is not sufficient to run the test case. You go back to step 3 to rework on the code.

If the test passes, the code meets the unit test controls, and can now proceed to step 5 for refactoring.

<h2>Step 5: Refactor Code</h2>
Refactoring is an activity where you make the code efficient. You do not change the functionality, but rather alter the internal dynamics to enhance the performance. Some examples include identifying and removing duplicate and dead pieces of code, simplifying nested classes and consolidating conditional expressions.

The code that has passed the unit test in step 5 is put through the process of refactoring – where the code is made efficient.

After refactoring, the developer picks up the next piece of unit test case – say the username field in the login page he is developing. He writes the unit test case for the username field and repeats the steps 1 through 5. The process iterates until the feature or software is fully developed.

<h1>Why should you Employ TDD Methodology?</h1>
There are primarily two major benefits of using TDD. 
<br /><br />
First, the code that comes out of this process is clean – in the sense that it is free from bugs, as testing guides and hand-holds the development lifecycle. It is an effective arsenal against defects – which is one of the major problems that plague the software development industry. I am not implying that we are going to 100% defect free code, but the defect rate is far lesser than what it would have been otherwise.
<br /><br />
Code coverage is a measure that we use in software development to identify the percentage of code that is tested. If every single line of the code is covered under the test cases, then the code coverage measured will be 100% - which is the target in ideal conditions. This is a necessary evil because you don’t want to introduce any lines of code that haven’t been scrutinized yet. However many organizations insist on a lower number ranging between 90-95% of code coverage. Tools such as SonarQube and Atlassian Clover help you in the code coverage measurement.
<br /><br />
With TDD however, it is possible to achieve 100% code coverage because you first write the test case, and write just enough code to make the test case pass.

<h1>Are there any Downsides in TDD Methodology?</h1>
In the earlier section, I took you through the process involving TDD. Now, let us see who all are involved in the TDD methodology – represented in figure 2.


<center>![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/5fe7055e-984e-4fec-9735-820fc5e78bc4.jpg)
<br /><br />
Figure 2: TDD Role Play</center>

In a project employing TDD, typically a business analyst will get the requirements from the customer or a product owner and pass it onto the developer. The developer based on his understanding of the requirements creates unit test cases and then writes the code. Once the test execution is successful, the developer will refactor the code and repeat the TDD process for the tasks and user stories assigned to him.
<br /><br />
The drawback in the process is in the understanding of the requirements. Suppose the business analyst explains that the customer’s requirement is to develop an account management system, the developer can possibly misinterpret the requirements and develop something totally different – such as a customer relationship management system.
<br /><br />
As you can see in the roleplay diagram, there is no tester involved. The developer writes the test case, and he also develops the code. So, there is no validation of the requirements throughout the cycle – which usually happens when testers are involved.
<br /><br />
Swaying away from the requirements is a costly mistake, and often ends up in plentiful of rework – which costs money, delays project timelines and angers the customer. This according to me is the major downside of the TDD methodology.
<br /><br />
There are other alleged drawbacks such as the process taking too much of effort and time, and the unit tests look to concentrate on the individual classes and functions rather than the overall user stories and customer requirements on a whole. I agree that the unit test cases work at a micro-level and does not holistically look at the requirements – which is fixed with the methodologies that I am going to introduce in the next section. On the topic of TDD taking up too plenty of effort and time, I believe that there is value in writing the test cases first, and achieving clean code and code coverage – which overhauls the effort and time needed to achieve it.

<h1>Alternatives to TDD</h1>

Some of the drawbacks that I discussed earlier are considered in drawing up the upgraded version of TDD. In fact, there are two of them. There are a few, but the prominent ones among them are ATDD and BDD.

ATDD stands for Acceptance Test Driven Development. In this methodology, there are a couple of major improvements over TDD. First, instead of writing unit test cases, acceptance test cases based on user stories are written, and then the code is developed. Second, the test cases are written by the tester, and the code is developed by the coder – which brings in a certain amount of validation for the requirements that are passed on from the business analyst.

However, it was observed that ATDD still had some drawbacks. What if both the tester and the coder understand the requirements incorrectly? We need another layer of control and validation. This layer is provided by the Behavioral Driven Development (BDD) methodology.

In BDD, acceptance test cases are developed, but there is a significant addition to the way requirements are obtained. The business analyst is expected to provide requirements along with relevant examples for each one of them. Through the use of examples, the tester and the coder can assimilate the requirements considerably better.

The next major variation in BDD is that the test cases are not written in the usual scripting language. Rather forms of natural languages such as Gherkin is employed. Using a natural to write acceptance test cases will equip the business analyst, product owners and customers to validate against the requirements.

Tools such as Cucumber and JBehave support BDD methodology.

Today most projects that are running on TDD are switching over to BDD. BDD is attractive because of the ease in which the acceptance test cases can be prepared and it acts as a natural bridge between the development teams and the customers. The proximity between the two enhances communication exchanges and reduces friction coming out of black boxes and delays.

Although BDD stands on its own today, it has taken its roots from the TDD methodology. The principles of TDD which started several years ago still drives BDD.
