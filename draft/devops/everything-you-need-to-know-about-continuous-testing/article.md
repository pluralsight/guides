# Start writing your guide here.

The fast-paced market conditions have made it impossible for rigidity and holism to take root in the IT industry. Especially in the software development industry, waterfall styled projects are dusking while Agile and DevOps (inspired) methodologies have been embraced.

<strong>Agile and DevOps: The New Way</strong>

Agile project management is well-documented, and there is plenty of literature on the processes, case-studies and other artifacts to use as guidance. However, DevOps is a different ball game altogether. It is a mystery capsule that can take on different forms based on the subject. Although the concept of DevOps has been documented, there is much more to it than meets the eye.

Remember that DevOps has two parts to it – Development and Operations – that work together to create synergy and a whole lot of other things. It is similar to yin and yang, where contradicting and conflicting priorities complement each other when unified. 
Dev Processes of DevOps

Four processes provide clarity and insight into the world of development within DevOps. They are the buzz words that light up professional circles, but the devil in the details is most often ambiguous. The three processes are:

•	Continuous Integration

•	Continuous Testing

•	Continuous Delivery

•	Continuous Deployment

In this post, I will briefly touch upon Continuous Integration and dive right into Continuous Testing.

<strong>Brief Introduction Continuous Integration</strong>

Continuous Integration is a process where developers integrate their code with the source code repository on a regular basis multiple times a day (if feasible), and whenever they integrate, the entire mainline is built, and other activities such as unit testing and checking the code-quality also take place. For example, if a project has four developers, and let’s say that each developer integrates his code three times a day, then there will be twelve integrations every day. This translates to twelve builds of the mainline, twelve sets of unit tests and the code quality checks that run for twelve times.

Figure 1: Continuous Integration Process

Figure 1 illustrates the process of Continuous Integration.

The benefit of integrating code frequently and running builds every time is to catch the defects in its budding stages rather than later – say UAT. By catching defects early, we reduce the risks associated with rewriting a large piece of code as opposed to smaller bits and controlling the integration in smaller chunks. If something was to go bad, a rollback would only involve a few hundred lines as opposed to a whole lot more.

When each of the integrations is successfully built, unit tested and code-quality checked, it goes into the next stage in the delivery pipeline called as continuous testing.
Testing, Feedback, and Defects

Before I jump into Continuous Testing, let us touch base on what testing is, and how and when it plays a crucial role.

Testing is a process where you validate the developed product against functional and non-functional requirements. Testing is the phase where you find out if the product actually works as per the design, and whether the product fulfills the user stories that have filled your product backlog.

In other words, testing phase provides you the feedback, which will help you determine if further changes are needed. Faster the feedback, it is better for you – why – because by going down the wrong road too far, you risk having to rework more than you originally developed. In some cases, it may not be possible to fix the defects because the feedback came in two days before your scheduled deployment. So, what do you do? You push the defects into production, hoping to fix it someday. Will defects always get fixed in production? You know the answer!

<strong>What is Continuous Testing?</strong>

Continuous Testing is the process in which the code integrations that are built during the Continuous Integration process get into a pipeline of various tests (integration, system, performance, regression and user acceptance to name some), and the tests get executed automatically – with absolutely zero human intervention.

Figure 2: Continuous Testing Process

Figure 2 illustrates the Continuous Testing process. Whenever developers integrate their code, it runs through the Continuous Integration cycle. If the process is successful, the binary that is built out of the Continuous Integration process will be tested automatically. As illustrated in figure 2, the binary is automatically run through integration test. If successful, system test follows. If successful, regression test follows. If successful automatic user acceptance test (UAT) is executed. Likewise, you can add, delete or modify any tests that you need to this pipeline. 

The process of Continuous Testing ensures that once the code is integrated, it gets tested automatically.

<strong>Difference between Automated Testing and Continuous Testing</strong>

Automated testing is a process where the execution of tests is run through the testing tools, and these tools are fed with testing scripts to run the test. There is no human intervention in the automated testing process, other than writing the test scripts.

Continuous Testing is a process where the execution of tests is run through the testing tools automatically following code integration, and the testing tools are pre-loaded with the testing scripts. There is no human intervention in Continuous Testing as well, other than with the script development.

So, what is the difference between automated testing and Continuous Testing?
In automation testing, the code is integrated with the mainline, automation test scripts developed and then the binaries are tested against these scripts automatically using automation test suites.

In Continuous Testing, the automation test scripts are written before the coding begins. So, when the code is integrated, the automation tests are automatically run one after another – hence the term Continuous Testing. Development methodologies such as TDD and BDD gel well with DevOps framework as they are one of the enablers for Continuous Testing.

The advantage of Continuous Testing over automation testing is that once a code is checked into the source code repository, the process to build and validate begins, and the feedback is obtained rapidly. There is no gap in the process where the integration, testing, and feedback mechanism await human intervention.

The difference between automated testing and Continuous Testing can be best explained in figure 3.

Figure 3: Automated Testing vs Continuous Testing

<strong>Do Testing the Continuous Way</strong>

The objective of DevOps is to deliver software efficiently to the customer. And you cannot deliver it quickly if you don’t get the feedback on time. By employing the Continuous Testing process, you can be rest assured that the feedback is rapid, and you can save tons of time in not redoing your code.
Developers and testers have often asked me whether writing all the test scripts before you begin to code is feasible and practical. The answer is yes, this is the essence to achieving Continuous Delivery and Continuous Deployment. Many organizations are able to deploy multiple times a day. There are reports that suggest that Amazon deploys five times a minute, that’s three hundred deployments an hour. What are they doing? Continuous Testing.
