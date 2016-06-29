# Leveraging a S3 bucket to store data to use in an Amazon Skill

As we continue to expand the capabilities of skills written as a Lambda function, it's key to start thinking ahead to how this code can be maintained.  This guide articulates how to take the [introductory skill from amazon] (https://github.com/amzn/alexa-skills-kit-js/blob/master/samples/reindeerGames/src/index.js) and expand it into a more intermediate exercise.

** The Problem **

Writting code that can be easily maintained is around keeping it as simple as possible.  The tutorials around writing a trivia skill are quite good and help the novice quickly get something up in running in an hour, but can be challenging to maintain over time.  As you add more trivia questions to keep the user interested, you may find that the nodeJS file becomes a little bloated, and runs into several hundred lines of code, much of which is in storing the various trivial questions for the function to rotate through.

The array at the beginning of the Reindeer Trivia skill looks like this...

    /**
    * This sample shows how to create a simple Trivia skill with a multiple choice format.
    * The skill supports 1 player at a time, and does not support games across sessions.
    */
    
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
        ...
        
When the initial welcome utterance is invoked, some initialization is done for the .  Here's the actual code snipet.

    function getWelcomeResponse(callback) {
    
    var sessionAttributes = {},
        speechOutput = "I will ask you " + GAME_LENGTH.toString()
            + " questions, try to get as many right as you can. Just say the number of the answer. Let's begin. ",
        shouldEndSession = false,

        gameQuestions = populateGameQuestions(),
        
The actual loading of the array occurs in the following function

    function populateGameQuestions() {
        var gameQuestions = [];
        var indexList = [];
        var index = questions.length;

        if (GAME_LENGTH > index){
            throw "Invalid Game Length.";
        }

        for (var i = 0; i < questions.length; i++){
            indexList.push(i);
        }

        // Pick GAME_LENGTH random questions from the list to ask the user, make sure there are no repeats.
        for (var j = 0; j < GAME_LENGTH; j++){
            var rand = Math.floor(Math.random() * index);
            index -= 1;

            var temp = indexList[index];
            indexList[index] = indexList[rand];
            indexList[rand] = temp;
            gameQuestions.push(indexList[index]);
        }

        return gameQuestions;
    }
    
If we want to split out the questions from the actual Lambda function, we can rewrite this function to pull the questions from an object in an S3 bucket.  While it makes this function far more complex, it has the benefit of separating out the working syntax of the skill from the data itself.  Here's one way of doing this.

    function populateGameQuestions() {
        // first start by creating an S3 object
        
        // these are the parameters - first the bucket name, then the data object
        
        // we do a 'get' to pull the object
        
        // now the code starts looking very similar to the original function
        
Once we've separated out this data, we can then routinely add more questions, and have the array span into hundreds of lines of syntax without ever changing the .  The investment in getting

** Setup **

There is some work to do beyond.  Remember in a serverless architecture roles are critical, and we need to modify the policy for the role that serves to execute the Lambda function to be entitled to read the S3 bucket.