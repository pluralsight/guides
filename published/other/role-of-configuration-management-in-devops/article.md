DevOps has several components that must work in unison for the team to meet its objectives. A key element, which usually serves as the center of the DevOps "machinery," is **configuration management**. 

<h1>What is Configuration Management?</h1>

Configuration management represents the one true source of the configuration items. A configuration item is anything that can be configured and that is absolutely necessary for the success of your project. For example, source codes, property files, binaries, servers, and tools can all be configuration items for a software firm.

In fact, the term configuration management means something or another in most industries. If you pick the automotive industry, for example, configuration management could refer to the configuration of the assembly line machines that are instrumental to manufacturing. Another configuration item could be the inventory of various automobile parts, as this would be configured early on in the development process.

> In software development and management, configuration management refers to the items that need to be configured and managed in order for the project to be successful. 

Configuration management, however, can have many connotations, depending on who's discussing it. Most people in the software development industry refer to the source code management alone as configuration management. But, there is a whole lot more to configuration than managing source codes when it comes to DevOps. This article discusses configuration management end-to-end and studies its role in DevOps.

<h2>The Process of Configuration</h2>

The configuration management process takes into consideration the following broad-set activities:
1.	<strong>Configuration identification</strong> &mdash; The configuration that needs to be maintained first must be identified. It can either be a manual process, such as maintaining the source code on a common repository or using discovery tools to automatically identify the configuration.
2.	<strong>Configuration control</strong> &mdash; Once the configuration items are identified, there is no guarantee that it will remain unchanged. In all probability, the configuration is likely to change. Thus, there needs to be an effective mechanism to control the changes that go into the configuration management system. In most configuration management frameworks, the *change management process* acts as a guardian for controlling the changes to the configuration management system.
3.	<strong>Configuration audit</strong> &mdash; Despite there being control mechanisms protecting against changes in configuration, means of bypassing exist. Therefore, *configuration audits* are necessary to keep an eye on configuration compliance.

<h1>Configuration Management in DevOps</h1>

DevOps spans across software development and operations phases. Therefore, it is only fitting that configuration management spans across both the areas. 

To define configuration management in DevOps, the inspiration comes from the book *Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation*, authored by Jez Humble and David Farley.

Configuration management in DevOps is coined as "comprehensive configuration management", and is made up of 
1.	<strong>Source Code Repository</strong> &mdash; Used primarily during the development phase.
2.	<strong>Artifact Repository</strong> &mdash; Used during the development and operations phases.
3.	<strong>Configuration Management Database</strong> &mdash; Used during the development and operations phases.

<center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/3c66133a-deaa-47a2-8a92-d90df37eee76.jpg)
<br />
Figure 1: Comprehensive Configuration Management</center>

<h2>Source Code Repository</h2>
A source code repository is the primary container for all versions of code. Apart from keeping all the code, it generally stores test scripts, build scripts, deployment scripts, and configuration files as well.

Some projects also leverage a source code repo to store binaries. However, this is not recommended because, depending on the number of builds, there can be numerous binaries, and they need to be stored in a different manner, as I will discuss when talking about Artifact Repositories.

So, what can be stored in a source code repository? Simply put &mdash; anything that is human readable goes into the source code repository. Software binaries are not human readable, so they get omitted. Test scripts, actual code, and config files are all readable (and usable) by humans, so they are included.

<h3>Types of Source Code Repositories</h3>
Not all source code repositories are the same. The times have changed and so has the technology. The source code repository technology of yesteryears is the **Centralized Version Control System** (CVCS). However, many software development projects still utilize this somewhat-antiquated technology.

The concept around CVCS is that the source code resides in a central server. The code needs to be checked out, changes committed and checked back into the repository. All these activities are performed directly onto the central repository.

The source code repository technology of today is **Distributed Version Control System** (DVCS). The source code in this technology not only resides on a central server but also on *all terminals used for development*. In other words, every developer will have the entire source code repository on his or her workstation. Any merge or change is performed locally and usually seamlessly.

<h3>Benefits of DVCS over CVCS</h3>
The greatest advantage of DVCS over CVCS is its availability. For DVCS to function, you don’t need an active network connection, but for CVCS it is absolutely necessary. If server access is blocked, development comes to a complete standstill. Thus, CVCS has a **single point of failure** (SPOF), something that goes against the principles of DevOps, which emphasize incessant development and maximum efficiency.

Since source code is available locally in DVCS, accessing, merging and pushing the code is much faster relative to CVCS.

DVCS also enables software developers to exchange code with other developers before pushing it to the central server and, consequently, to all other developers.

In fact, there are no notable disadvantages with DVCS other than perhaps the storage needs on a developer’s terminal if source code contains an elongated history of changesets.

<h3>Toolsets for Managing Source Code</h3>

There are a number of source code repos available off-the-shelf today. Git's repo system as well as Mercurial are key DVCS technologies. Git is perhaps the most popular VCS platform, likely owing to its flexibility in allowing multiple simultaneous changes. (No more check-out and check-in by individual developers.)

The Git software can either be used on premises or hosted on public clouds under various guises (i.e. Bitbucket, Github and Gitlib.)

Meanwhile, some CVCS source code repositories still in vogue include Subversion and CVS. 

Given the rise of DVCS and its abundant advantages, organizations have made a push to migrate CVCS toolsets to DVCS.

<h2>Artifact Repository</h2>

An **artifact repository** is a database for storing binaries. Additionally, test data and libraries can be stored on it as well. Whereas source code repository is meant to store human readable files, an artifact repository stores machine files.

<h3>Continuous Integration</h3>

Principles of **Continuous Integration** encourage developers to push code to the main line frequently. Each push triggers a build, which results in a binary getting generated. Depending on the size of the project, the artifact repo could end with thousands of binaries.

<h3>Management of Artifacts</h3>

Binary management can be cumbersome; with thousands of binaries, which one should the release manager choose to push into production? Believe me, this is an undesirable task. To the release manager's aid, an artifact repository help big time. 

How? The artifact repo comes with two logical partitions for storing and managing binaries:
1.	**Snapshot**
2.	**Release**

Every time a build is successfully run, the binary that gets generated is stored in the Snapshot repository. But not all of them get pushed into production unless the project adopts the Continuous Deployment methodology. The binary that gets pushed into the production is first moved into the Release partition before getting deployed into production. (See Figure 2.)

<center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/5bb8f8af-c70c-4211-a091-6a57e91a1180.jpg)<br />
Figure 2: Artifact Repository</center>

The figure demonstrates that various binaries are generated every time the build is successful. The binaries in this example are named as Binary 0.x, and are stored in the Snapshot partition. Not every binary gets promoted into production. 

The binaries that are promoted get moved into the Release partition from the Snapshot partition. Examples: Binary 0.3 and Binary 0.5. 

**This is key to the release management process.** Let’s say that the Binary 0.5 is deployed into production, and the deployment fails. As a fallback step, the deployment must roll back to the previous version. 

The previous binary versions used are stored in the Release partition, making planning and executing releases efficient. 

Without a logical partition, imagine the planning and effort it would take to identify the previous version in production and roll it back.

<h2>Configuration Management Database</h2>

The **Configuration Management Database** (CMDB) comes from the Information Technology Infrastructure Library (ITIL) service management framework. The CMDB is a repository of various infrastructure devices, applications, databases, and services. Not just an inventory, it also bears relationships between the various elements within the CMDB, as illustrated in figure 3. 

<center>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/db096003-a8e3-4bb2-a0bb-9350b2dd871f.jpg)

<br /><br />Figure 3: CMDB Illustration</center><br />


In the figure above, services, applications, databases and servers are represented by the different-colored boxes. In the illustration, Service 1 depends on Application A, as seen by the arrow relationship. Application A leverages on Database 1. Both the Application A and Database 1 reside on Server A.

In a real CMDB, however, the arrows mean something different from the relationships as described. For example, an application generally *makes use of* a database &mdash; this is the relationship between the two entities. An application residing on a server leads to a *dependent on* relationship. The relationship around data flow between applications (indicated between Application B and Application C) is one of *data dependency*.

<h3>CMDB for Change Management</h3>

The CMDB is particularly useful when you are trying to make a change to any of the applications, databases or servers. 

Let’s say you want to make a change to Application B. To make the change, you must first do an impact assessment. CMDB helps in performing impact assessments, and in the illustration, suppose changes are done to Application B. The impact assessment will read that any changes done to Application B will impact Application C, as the data is flowing through it.

Today, software development seldom happens in isolation. The software to be developed is either an improvement over existing code or is getting plugged into an enterprise network of applications. Therefore, it is critical that the impacts are assessed to the tee, and CMDB is a great help in this area.

<h3>CMDB for Provisioning Environments</h3>

Another application of CMDB in DevOps is in environment provisioning. Today we can spin up environments on the go by tools like Ansible in our scripts. When you key in the exact configuration of the production server, the environment provisioning tools create a prod-like server in a snap of a finger. 

But how is it that you are going to obtain the complete configuration of a server? The most straightforward way is to refer to a CMDB.

Let’s say Server C is a production server that needs to be replicated. In the CMDB, the server C entry will provide the complete configuration of the server, which is a great help in scripting provisioning scripts such as Playbooks (compatible with Ansible.)

<h3>CMDB for Incident Management</h3>

The CMDB also has other benefits such as supporting the incident management teams during incident resolutions. The CMDB readily provides the architecture of applications and infrastructure, which is used to troubleshoot and identify the cause of the issue.

<h1>DevOps Starts and Ends with Configuration</h1>
Yes, it is true that you cannot really do DevOps without configuration management in place. I have shared the principles and examples around the comprehensive configuration management, without which the artifacts and other useful information will be all over the place, and in a disorganized manner.

Remember the objective of DevOps &mdash; developing software as quickly as possible. This objective can only be done through proper organization and planning, and the comprehensive configuration management gives you sufficient ammo to power up the DevOps machine.

For this reason, configuration management professionals are in great demand. As a solution architect in configuration management, I don’t claim to know it all; every project throws new challenges in my direction, thus heralding new findings and learnings. For me, this is the beauty of configuration management.

____

I hope you found this guide on Configuration Management and DevOps interesting. Please favorite this article and drop comments, questions, and feedback in the discussion section below. Thank you for reading!