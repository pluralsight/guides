DevOps is made of several elements that need to work in unison for the DevOps machinery to meet the objectives that it is set out to do. One of the key elements, which sits right in the center is the configuration management. 

It is a critical success factor – if you get it right, there is a good chance that you will end up meeting all the DevOps objectives. Other elements that make up the DevOps jigsaw are reviews, testing (functional and non-functional) and environment provisioning among others.

Configuration management can have many connotations, depending on who you are talking to. Most people in the software development industry refer to the source code management alone as configuration management. But, there is a whole lot more to configuration than managing source codes in DevOps, and this article delves into the end to end configuration management and the role it plays in the DevOps machinery.

<h1>What is Configuration Management?</h1>
Configuration management represents the one true source of the configuration items. A configuration item can be anything that can be configured, and is absolutely necessary for the success of your project. Examples of a configuration item in a software industry include source codes, property files, binaries, servers, and tools.

The term configuration management is used in all industries. So, it may mean different elements in the respective industries. 

In you pick the automotive industry, configuration management refers to the configuration of the assembly lines, the machines that are instrumental in the manufacturing process, and also the inventory of the various parts that go into making of an automobile.

Focusing back on our industry - software development and management, configuration management refers to the items that need to be configured and managed in order for the project to be successful. I will go into the details of configuration management in the software industry in the next section – configuration management in DevOps.

<h2>The Process of Configuration</h2>
No matter the industry, the configuration management process takes into consideration the following broad-set activities:
1.	<strong>Configuration identification</strong> – The configuration that needs to be maintained first needs to be identified. It can either be a manual process such as maintaining the source code on a common repository or using discovery tools to automatically identify the configuration.
2.	<strong>Configuration control</strong> – Once the configuration items are identified, there is no guarantee that it will remain unchanged. In all probability, the configuration is likely to change. So, there needs to be an effective mechanism to control the changes that go into the configuration management system. In most configuration management frameworks, the change management process acts as a guardian for controlling the changes to the configuration management system.
3.	<strong>Configuration audit</strong> – Even when there are controlling mechanisms for changes to the configuration, it does not necessarily mean that you cannot bypass it. Therefore, configuration audits are a necessary evil to keep an eye on configuration compliance.

<h1>Configuration Management in DevOps</h1>
DevOps spans across software development and operations phases. Therefore, it is only fitting that the configuration management spans across both the areas (unlike source code management as some people believe).

To define configuration management in DevOps, the inspiration comes from the book Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation, authored by Jez Humble and David Farley.
Configuration management in DevOps is termed as comprehensive configuration management, and is made up of 
1.	<strong>Source Code Repository</strong> –Used primarily during the development phase.
2.	<strong>Artifact Repository</strong> – Used during the development and operations phases.
3.	<strong>Configuration Management Database</strong> – Used during the development and operations phases.

<center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3c66133a-deaa-47a2-8a92-d90df37eee76.jpg)

Figure 1: Comprehensive Configuration Management</center>

<h2>Source Code Repository</h2>
A source code repository primarily houses all versions of code. Apart from the code, it is a good practice to store test scripts, build scripts, deployment scripts and configuration files in a source code repository.

Some projects also leverage a source code repository to store binaries. This is not recommended, as the binaries are numerous depending on the number of builds, and they need to be stored in a different manner – which I will discuss under the section – Artifact Repository. 
So, what can be stored in a source code repository? Here’s an easy way to remember it - anything that is human readable goes into the source code repository. This implies: software binaries are not human readable – so it does go into it.

<h3>Types of Source Code Repositories</h3>
Not all source code repositories are the same. The times have changed, and along with it the technology. The source code repository technology of yesteryears is the centralized version control system (CVCS). Surprisingly, many software development projects still leverage on this antiquated technology.

The concept around CVCS is that the source code resides in a central server. The code needs to be checked out, changes committed and checked back into the repository. All these activities are performed directly onto the central repository.

The source code repository technology of today is distributed version control system (DVCS). The source code in this technology not only resides on a central server, but also does on all terminals used for development. In other words, every developer will have the entire source code repository on his/her workstation, and any merge or other changes to be performed is done locally, and is generally seamless.

<h3>Benefits of DVCS over CVCS</h3>
The greatest advantage of DVCS over CVCS is its availability. For DVCS to function, you don’t need an active network connection, and for CVCS it is absolutely necessary. If there are obstructions to accessing the central server, the entire development activity comes to a standstill. So, CVCS can be a single point of failure (SPOF), which goes against the principles of DevOps – where we like things to keep moving all the time in order to deliver software at the earliest possible hour.

As the source code is available locally in DVCS, the whole activity around accessing, merging and pushing the code is faster compared to the CVCS counterparts.

It is also possible in DVCS that software developers can exchange their pieces of code with other developers to check if all is well, before pushing it to the central server, and to the rest of the developers.

There are no notable disadvantages with DVCS other than the storage needs on developer’s terminal if the source code was to contain an elongated history of changesets.

<h3>Toolsets for Managing Source Code</h3>
There are a number of source code repositories available off the shelf today. The Git source code repository, which falls into the DVCS bracket, is perhaps the most popular owing to its flexibility that allows multiple changes to be done simultaneously. So, no more check-out and check-in by individual developers.

The Git software can either be hosted on premises or is available on public clouds, under various guises – such as Bitbucket, Github and Gitlib.

Another source code repository built around DVCS worth a mention is Mercurial.

The source code repositories built around CVCS, and that are still in vogue include Subversion and CVS. 

There are a number of migration projects across organizations where projects that ran on CVCS toolsets are moving towards DVCS.

<h2>Artifact Repository</h2>
An artifact repository is a database meant primarily to store binaries. Apart from binaries, test data, and libraries can be stored on it. Just as source code repository is meant to store human readable files, an artifact repository is primarily used to store machine readable files.

<h3>Starts with Continuous Integration</h3>
In Continuous Integration, we encourage developers to push their codes to the mainline in short intervals. This in turn triggers build, which results in a binary getting generated. Depending on the size of the project, we could end up looking at an artifact repository that potentially has hundreds and thousands of binaries.

<h3>Management of Artifacts</h3>
The management of binaries can be cumbersome. When there are thousands of binaries, which one must the release manager select to be pushed into the production? Believe me, it is an unenviable task. To his/her aid, an artifact repository can be a big help. How? It comes with two logical partitions for storing and managing binaries:
1.	Snapshot and
2.	Release

Every time a build is successfully run, the binary that gets generated is stored in the Snapshot repository. But not every one of them gets pushed into production unless the project adopts the Continuous Deployment methodology. The binary that gets pushed into the production is first moved into the Release partition, and then it gets deployed into production. This is illustrated in figure 2.

<center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/5bb8f8af-c70c-4211-a091-6a57e91a1180.jpg)
Figure 2: Artifact Repository</center>

<h3>Illustration on Artifact Repository</h3>
In the figure 2 illustration, various binaries are generated every time the build is successful. The binaries in this example are named as Binary 0.x, and are stored in the Snapshot partition. Not every binary is promoted into production. 

The binaries that gets promoted into production are moved into the Release partition from the Snapshot partition. Examples: Binary 0.3 and Binary 0.5. 

This is a key step in the release management process. Let’s say that the Binary 0.5 is deployed into production, and the deployment fails. As a fallback step, the deployment must rollback to the previous version. 

The previous binary versions used are stored in the Release partition, and it makes planning and executing releases efficient. 

In the absence of a logical partition, imagine the amount of planning that had to go in for identifying the previous version that was put on production, and the amount of management needed to execute the rollback.

<h2>Configuration Management Database</h2>
The configuration management database (CMDB) comes from the ITIL service management framework. The CMDB is a repository of various infrastructure devices, applications, databases and services. It is not just an inventory, but it also bears relationships between the various elements within the CMDB. This is illustrated in figure 3.

<center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/db096003-a8e3-4bb2-a0bb-9350b2dd871f.jpg)

<br /><br />Figure 3: CMDB Illustration</center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f7f2c245-a8ef-4c5b-b1e9-4d71951c5dd1.jpg)


In the figure 3, I have represented services, applications, databases and servers. In the illustration, Service 1 is dependent on Application A. This is represented through the relationship (arrow in the figure). The Application A leverages on Database 1. Both the Application A and Database 1 reside on Server A. 
All these relationships are indicated with arrows. However, in a real CMDB, each of these arrows have a different connotation. For examples the relationship between an application and a database is generally makes use of. To indicate the application residing on a server, the relationship can be dependent on. The relationship around data flow between applications (indicated between Application B and Application C) is data dependency.

<h3>CMDB for Change Management</h3>
The CMDB is particularly useful when you are trying to make a change to any of the applications, databases or servers. Let’s say you want to make a change to Application B. To make the change, you must first do an impact assessment. CMDB helps in performing impact assessments, and in the illustration, suppose changes are done to Application B. The impact assessment will read that any changes done to Application B will impact Application C, as the data is flowing through it.

Today, software development does not happen in isolation. The software to be developed is either an improvement over an existing one, or is getting plugged into an enterprise network of applications. Therefore, it is critical that the impacts are assessed to the tee, and CMDB is a great help in this area.

<h3>CMDB for Provisioning Environments</h3>
Another application of CMDB in DevOps is in environment provisioning. Today we can spin up environments on the go through scripts using tools such as Ansible. When you key in the exact configuration of the production server, the environment provisioning tools create a prod-like server in a snap of a finger. 

But how is it that you are going to obtain the complete configuration of a server? The most straightforward way is to refer to a CMDB.

Let’s say Server C is a production server that needs to be replicated. In the CMDB, the server C entry will provide the complete configuration of the server, which is a great help in scripting provisioning scripts such as Playbooks (compatible with Ansible).

<h3>CMDB for Incident Management</h3>
The CMDB also has other benefits such as supporting the incident management teams during incident resolutions. The CMDB readily provides the architecture of applications and infrastructure, which is used to troubleshoot and identify the cause of the issue.

<h1>In DevOps, it Starts and Ends with Configuration</h1>
Yes, it is true that you cannot really do DevOps without configuration management in place. I have shared the principles and examples around the comprehensive configuration management, without which the artifacts and other useful information will be all over the place, and in a disorganized manner.

Remember the objective of DevOps – developing software as quickly as possible. This objective can only be done through proper organization and planning, and the comprehensive configuration management gives you sufficient ammo to power up the DevOps machine.

From a role perspective, professionals who are configuration management are in great demand. I am a solution architect in configuration management, and I don’t claim to know-it-all. Every project throws up new challenges at me, which heralds new findings and learnings – which is the beauty of configuration management.