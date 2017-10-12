
A week into writing functional tests for the web related work I ran into this weird issue of not getting chrome up and running for getting my WebdriverIo and Nightwatch tests to work. The issue was quite flaky. 
When I would run them on my local it would run just fine because it was a configure once run all the time scenario.
The real problem came when I started putting it in the CI / CD pipeline. The slaves were spawned as containers on AWS. It was at this point I realised that either google chrome could not be launched or the chrome-driver had to be updated.


After a week of enduring flaky tests and broken pipelines, the worst hit came when the tests actually failed and I looked at the radiator and simply assumed that the tests had failed because of the whole chrome issue and went ahead and deployed it anyway.


The other option was Browserstack and or Saucelabs which … are a bit expensive given the fact that i was not even looking at cross-browser testing at this point in time.

It was it this point I started googling out alternatives and came across this wonderful solution. If you would rather concentrate on the functional tests, rather than the whole infrastructure of getting it to run, you would also spare yourself the agony of venting out the whole frustration of it.

Enter `Elagu / selenium`


This essentially spawns up a selenium grid that has the latest versions of chrome and firefox installed. The code and the configuration that you have now need not go through the pain of setting up chrome and then ChromeDriver and then gluing the pieces together to make sure the tests can be made to work.


Now that you have the pieces, you can simply create a pipeline script in Jenkins that
pulls in the latest docker image , starts of a selenium grid and you can juts point to `localhost:4444` to have your tests up and running in seconds

You can even spice it up with having video records of the tests:
Much like browserstack and saucelabs.


Sample pipeline script:

```
pipeline {
    agent {
        label 'Your Jenkins agent'
    }
    stages {
        stage('Pull latest selenium docker') {
            steps {
                slackSend (color: '#0000FF', message: "START: Pipeline '${env.JOB_NAME} - [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
                sh 'docker pull elgalu/selenium'
            }
        }
        stage('Run selenium grid') {
            steps {
                sh 'docker run -d --name=grid -p 4444:24444 -p 5900:25900 -e TZ="US/Pacific" -v /dev/shm:/dev/shm --privileged elgalu/selenium'
            }
        }
        stage('Wait 90 seconds for it to boot up') {
            steps {
                sh 'docker exec grid wait_all_done 90s'
            }
        }
        stage('Check if docker is up') {
            steps {
                sh 'wget --retry-connrefused --no-check-certificate -T 30  http://localhost:4444/grid/console -O /dev/null'
            }
        }
        stage('Pull the repo') {
            steps {
                git 'fooo'
            }
        }
        stage('run tests') {
            steps {
               // Command to run the tests go here
            }
        }        
    }
    post {
        always {
            sh 'docker rm -vf grid'
        }
    }
}
```