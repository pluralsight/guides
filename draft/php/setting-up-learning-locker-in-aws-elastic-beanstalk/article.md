 

AWS Elastic Beanstalk (EB) is both a dream and a nightmare. A dream when you have your apps running there, and you can enjoy seamless load balancing and autoscaling, automated versioning, and a lot of other very nice options, that reduce the hassle of maintaining a web application. A nightmare when you are setting things up, almost no specific documentation can be found, and errors and problems abound.

In theory, most web applications can be deployed to EB. In practice, it can be a lot of work, defining how to configure scripts and dependencies.
In this article, I’ll add my two cents, and try to document how to set up Learning Locker, the Open Source Learning Record Store (https://learninglocker.net/). I hope it helps people who want to set up their own LRS.

By the way, I am not affiliated with HT2 or Amazon. My opinions here are my own and, while there may be different implementations to this project, this is the way I finally made it work. I do work in the eLearning field, at SHIFT eLearning (https://shiftelearning.com), where I am CTO.

So, without further ado, let's begin.

# Requirements

- An Amazon Web Services account (https://aws.amazon.com).
- Knowing your way around CLI tools. While most of this can be done using the web client, I find it a lot faster and easier to just work on the terminal. Also, this is more scalable, and can be done even on remote servers via SSH.
- AWS CLI tools running on your development terminal: (https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html?icmpid=docs_elasticbeanstalk_console).
- Access to a running MongoDB database. I won’t explain how to set that up here (maybe in a future article). For testing pourposes you can use mLab (https://mlab.com), or any other provider.
- Git, Vim (or any other plain text editor).

# Clone Learning Locker
Very simple:
`git clone git@github.com:LearningLocker/learninglocker.git`

# Add EB configuration files
- In your new learninglocker directory, create a new subdirectory called `.ebextensions`
- Inside that, create a new file: `learninglocker.config`. You can use any name you want, as long as it is inside `.ebextensions`, and ends with `.config`.
- Add the following to that file:

`commands:  
    01_installMongoExtension:  
    `

commands:
  01_installMongoExtension:
    command: 'printf "\n" | sudo pecl install -f mongo'
    ignoreErrors: false
    test: "php -r \"exit(extension_loaded('mongo') ? 1 : 0);\""
  02_updateComposer:
    command: export COMPOSER_HOME=/root && /usr/bin/composer.phar self-update

option_settings:
  - namespace: aws:elasticbeanstalk:application:environment
    option_name: COMPOSER_HOME
    value: /root
  - namespace: aws:elasticbeanstalk:php:phpini
    option_name: document_root
    value: /public

config_options:
    max_execution: 120

container_commands:
  01_optimize:
    command: "/usr/bin/composer.phar dump-autoload --optimize"
  02_artisanMigrate:
    command 'php artisan migrate'

- Lines 2 to 5 install the MongoDB extension that Learning Locker needs to communicate with the database.
- Lines 6 and 7 update Composer, just in case.
- Lines 10 to 12 set the COMPOSER_HOME variable.
- Lines 13 to 15 set the app’s public directory to /public. This sometines doesn’t work and you must set it up using the web interface.
- Lines 18 sets up PHP max_execution value. Change this to whatever you need. If using a free database provider, use a high number; they have a tendency to timeout.
- Line 22 optimizes your installation.
- Line 24 runs the database migration.

## Add a local database configuration file
Go to /app/config/local.
There, create a file named database.php, with the following contents:

This just makes PHP read all database connection variables from the environment. This is a lot more secure than hardcoding those values in files that will be commited to Git. This way also makes it easier to change them.
Set your encryption key
Go to /app/config/, and open app.php.
Look for this line:
'key' => 'yoursecretkey',
Here, you have two options. Use environment variables like before, or hardcode the key.
I recommend using environment variables, but that may cause some problems if you need to run scripts outside the app.
Anyway, if you shoose to use environment variables, change it to this:
'key' => $_ENV['ENCRYPTION_KEY'],
If you choose to hardcode it, just type the value you want. It should be a 32 chars string:
'key' => 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
Edit email relay configuration
Open /app/config/mail.
Here we will be using environment variables again. Change it to something like this:

It’s basically the same we have been doing; changing variable asignment from hardcoded to environment. Make sure to set ‘pretend’ to false; otherwise Learning Locker won’t be sending any emails, even if everything else is right.
Initialize your Elastic Beanstalk application and environment
Go back to /, and run
eb init
Choose PHP 5.5 when asked, and answer all other questions according to your needs.
After that script is done, set up your environment:
eb create
or:
eb create --single
The first one creates a load balanced environment. The second one, sets up a single instance one. The former is best for production, the latter is best for testing.
Again, answer the questions according to your needs.
Deploy your app
When all that is done, you are ready to deploy Learning Locker. To do that you have to do two things.
First, commit all changes you just made:
git add .
git commit -m "Add Elastic Beanstalk support"
You must always commit changes to Git before deploying to EB. EB uses Git’s archive function, and all uncommited changes won’t be uploaded.
Finally, deploy:
eb deploy
This script takes a while to return. Go grab a cup of coffee or just stare at the screen.
Fill up all those environment variables
The last step is to assign values to the environment variables you created. That is done with a command like this:
eb setenv DB_HOST=<database host> DB_PORT=<database port> DB_USERNAME=<database username> DB_PASSWORD=<database password> DB_DATABASE=<database name> ENCRYPTION_KEY=<2 character random string> MAIL_HOST=<mail host> MAIL_USERNAME=<mail username> MAIL_PASSWORD=<mail password> MAIL_PORT=<mail port> MAIL_FROM_ADDRESS=<mail from address> MAIL_FROM_NAME=<mail from name>
Where the strings inside <> represent your actual values. For example,
eb setenv DB_HOST=mdb.example.com DB_PORT=27017 DB_USERNAME=locker DB_PASSWORD=s3cur3PWD DB_DATABASE=learninglocker ENCRYPTION_KEY=frkdwLeLOerpEw34FdE45FGVdele4r8o MAIL_HOST=smtp.mailgun.net MAIL_USERNAME=somebody MAIL_PASSWORD=anoth3rPWD MAIL_PORT=587 MAIL_FROM_ADDRESS=no-reply@example.com MAIL_FROM_NAME=LearningRecordStore
If you want to change the value of any variable, you just set it up without the others:
eb setenv MAIL_USERNAME=somebodyelse
Optionally use S3 for file storage
I hate it when applications store files on the hard drive, and don’t give you any options. This is a major problem when deploying load balanced apps, that do not share a common file repository.
LearningLocker, fortunately, is not one of them. Since verision 1.9 they allow you to use quite a few other services.
To use S3, you just need to set some more environment variables: FS_REPO, FS_S3V3_KEY, FS_S3V3_SECRET, FS_S3V3_REGION, FS_S3V3_VERSION, FS_S3V3_BUCKET and FS_S3V3_PREFIX.
Set them up as the ones vefore, with your own values. You should have already created a bucket, an user, set up access.
Notes
Because they want all Pull Requests to go to develop by default, clones their ‘develop’ branch. If you plan to deploy to production, you should checkout one of the production branches, right after cloning.
It’s been noted, that it is not ideal to modify the files in the /app/config folder, and that it’s better to create new ones in /app/config/local. I generally agree with that observation, but the problem I see is that will prevent you from running Learning Locker locally in your development computer (since the ‘local’ files would point to AWS resources).
Conclusion
That’s pretty much it. Those are the steps that, after a lot of trial/error/head-banging worked for me. Feel free to post any comments, if you have any questions, or if you’ve found a better way to do any of this.
Cheers!
References
http://blog.goforyt.com/laravel-aws-elastic-beanstalk-dev-guide/
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_PHP.container.html
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-setenv.html
http://docs.learninglocker.net/installation/
https://learninglocker.net/blog/load-balancing-learning-locker/