# Leveraging a S3 bucket for data in a Amazon Skill

As we continue to expand the capabilities of skills written as a Lambda function, it's key to start 


The Problem
-----------

Writting good code is around keeping it as simple as possible.  The tutorials around writing a trivia skill are quite good, however as you add more trivia questions, you may find that the nodeJS file becomes a little bloated, and runs into several hundred lines of code.

The array at the beginning of the Reindeer Trivia skill looks like this...

var questions = [
    {
        "Reindeer have very thick coats, how many hairs per square inch do they have?": [
            "13,000",
            "1,200",
            "5,000",
            "700",
            "1,000",
            "120,000"
        ]
    },
        {
        "The 1964 classic Rudolph The Red Nosed Reindeer was filmed in:": [
            "Japan",
            "United States",
            "Finland",
            "Germany"
        ]
    },