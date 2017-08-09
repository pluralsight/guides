Current fast-paced market conditions have made it impossible for rigidity and holism to take root in the tech industry. Particularly in software development, [waterfall-styled projects](https://en.wikipedia.org/wiki/Waterfall_model) are going unused after companies and developer teams have begun embracing Agile- and DevOps-inspired methodologies.

#### Prerequisites

I highly recommend reading up briefly on [Agile](https://en.wikipedia.org/wiki/Agile_software_development), [DevOps](https://en.wikipedia.org/wiki/DevOps), and other forms of project management before jumping into this guide! Some of the topics discussed here are much easier to understand with prior knowledge of terms. 

### <strong>Agile and DevOps: The New Way</strong>

Whereas Agile project management is well-documented -- there is plenty of literature on its processes, as well as case-studies and other artifacts to use as guidance -- DevOps is a different ball game. It is a mystery capsule that can take on different forms based on the subject. Although the concept of DevOps has been documented, there is much more to it than meets the eye. As such, this guide will focus more on the DevOps side of things. 

DevOps has two parts to it – Development and Operations. The synergy between these parts is similar to yin and yang in that contradicting and conflicting priorities complement each other when unified. 

### The Dev Side of DevOps

Four processes provide clarity and insight into the world of development within DevOps. They are the buzz words that light up professional circles, but the devil in the details is most often ambiguous. The three processes are:

•	Continuous Integration

•	Continuous Testing

•	Continuous Delivery

•	Continuous Deployment

In this post, I will briefly touch upon Continuous Integration and dive right into Continuous Testing.

# Continuous Integration

Continuous Integration is a process where developers integrate their code with the source code repository on a regular basis, multiple times a day (if feasible). Whenever they integrate, the entire mainline is built, and other activities such as unit testing and checking the code-quality also take place. 

For example, if a project has four developers, and let’s say that each developer integrates his code three times a day, then there will be twelve integrations every day. This translates to twelve builds of the mainline, twelve sets of unit tests and the code quality checks that run for twelve times.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7ac20ae9-9a83-4a3e-9a6d-9ad14a4317ca.jpg)

<center>Figure 1: Continuous Integration Process</center><br />

Integrating code frequently and running builds every time catches defects sooner rather than later. If something goes wrong earlier in the process, a rollback generally only involves a few hundred lines as opposed to a whole lot more restructuring. 

When each of the integrations is successfully built, unit tested and code-quality checked, the work goes into the next stage in the delivery pipeline called as continuous testing.

# Continuous Testing

### Testing, Feedback, and Defects

Before I jump into Continuous Testing, let us touch base on what testing is, and how and when it plays a crucial role.

Testing is a process where you validate the developed product against functional and non-functional requirements. Testing is the phase where you find out if the product actually works as per the design, and whether the product fulfills the user stories that have filled your product backlog.

In other words, testing phase provides you the feedback, which will help you determine if further changes are needed. Faster the feedback, it is better for you – why – because by going down the wrong road too far, you risk having to rework more than you originally developed. In some cases, it may not be possible to fix the defects because the feedback came in two days before your scheduled deployment. So, what do you do? You push the defects into production, hoping to fix it someday. Will defects always get fixed in production? You know the answer!

### What is Continuous Testing?

Continuous Testing is the process in which the code integrations that are built during the Continuous Integration process get sent into a pipeline of various tests (integration, system, performance, regression and user acceptance to name some), and the tests get executed automatically – with zero human involvement.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/1fc1f5d7-a117-46ee-b7d1-9c3a2eefa0ac.jpg)

<center>Figure 2: Continuous Testing Process</center><br />

Whenever developers integrate their code, it runs through the Continuous Integration cycle discussed above. If the process is successful, the binary that is built out of the Continuous Integration process will be tested automatically. As illustrated in figure 2, the binary is automatically run through integration test. If successful, system test follows. If that succeeds, regression testing follows. If regression succeeds, automatic user acceptance tests (UAT) are executed. Likewise, you can add, delete or modify any tests that you need to this pipeline. 

The process of Continuous Testing ensures that once the code is integrated, it gets tested automatically.

### Difference between Automated Testing and Continuous Testing

Automated testing is a process in which developer code is run through testing tools fed with testing scripts that run each test. There is no human intervention in the automated testing process, other than writing the test scripts.

Continuous Testing is a process where the execution of tests is run through the testing tools *automatically following code integration*. As before, the testing tools are pre-loaded with the testing scripts. There is no human intervention in Continuous Testing as well, other than with the script development.

So, what is the difference between Automated Testing and Continuous Testing?
In Automation Testing, the code is integrated with the mainline, then automation test scripts are developed, and then the binaries are tested against these scripts automatically using automation test suites.

In Continuous Testing, the test scripts are written before the coding begins. So, when the code is integrated, the automation tests are automatically run one after another – hence the term Continuous Testing. Development methodologies such as TDD and BDD gel well with DevOps framework as they are one of the enablers for Continuous Testing.

The advantage of Continuous Testing over Automation Testing is that once code is checked into the source code repository, the process to build and validate begins, and the feedback is obtained rapidly. There is no gap in the process where the integration, testing, and feedback mechanism await human intervention.

The difference between automated testing and Continuous Testing can be best explained in figure 3.




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b258fa6a-45d1-4490-8b21-fbb829e38f76.jpg)
<center>Figure 3: Automated Testing vs Continuous Testing</center> <br />

### Testing the Continuous Way

Did you know that traditionally testing has been considered as a bottleneck in the software development lifecycle? Developers complete the coding activity, and hand it over to testing folks. While the testers are busy at what they do best, developers often have to await feedback. DevOps came in with a single objective of cutting down the software delivery timeline, and to deliver a better quality software.
How can we cut down the development time and still achieve quality? The answer to this question is Continuous Testing. It cuts down the feedback cycle significantly, and this benefits in avoiding rewriting pieces of code.

Developers and testers have often asked me whether writing all the test scripts before you begin to code is feasible and practical. The answer is yes, this is the essence to achieving Continuous Delivery and Continuous Deployment. 

There are a number of organizations such as Amazon and Netflix who are able to deploy multiple times a day. They keep their deployments small, apply Continuous Testing and deliver unparalleled results. In this age, we have to move away from automated testing into Continuous Testing. This is the future, and the survival depends on how quickly organizations are able to transform.
