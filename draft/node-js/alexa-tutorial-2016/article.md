### Important information about how Alexa works.

Alexa is based on a question->response similar to Siri and Google Now services.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8908f30c-1e20-4de2-add1-8efa12f06544.png)

To create a skill we will need to **Create an Alexa skill** to capture the request like "Alexa, Ask HackGuides what time is it?"

This will then get translated from the Alexa Skill API into a **method call into the Amazon AWS Lambda Services** that is is then sent back to the user through the Alexa Skill.

![Alexa Question Response](https://developer.amazon.com/public/binaries/content/gallery/developerportalpublic/solutions/alexa/alexa-voice-service/images/avs_getting_started_1.png)



# Lets get started

To create a skill you will need to create for [developer.amazon.com:](https://developer.amazon.com/edw/home.html#/skills/lis): 

Now that you have an account you can begin to [Add a new Skill](https://developer.amazon.com/edw/home.html#/skills/list): 

![Alexa Skill Kit](https://raw.githubusercontent.com/pluralsight/guides/master/images/cc72e1e7-057d-48fa-9609-63fbe3f8da87.png)

This is the list of all your skills and you can click the top right button to create a new skill:
![Add New SKill](https://raw.githubusercontent.com/pluralsight/guides/master/images/f7e4d0bc-ae2d-428d-8622-931bd22f693b.png)


#### Skill Information

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/598b8c4b-a0c9-4c6d-bc4d-563797d8c97a.png)
- Update the "Name" with Hack Guides SKill Demo.
- Update the "Invocation Name" with Hack Guides

#### Interaction Model


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f35c7195-4064-408d-9c4a-e62ab30d54f1.png)


```{
  "intents": [
    {
      "intent": "GetNewFactIntent"
    },
    {
      "intent": "AMAZON.HelpIntent"
    },
    {
      "intent": "AMAZON.StopIntent"
    },
    {
      "intent": "AMAZON.CancelIntent"
    }
  ]
 }```
 

#### Configuration




Alexa has been made popular by the [Amazon Echo](https://www.amazon.com/Amazon-Echo-Bluetooth-Speaker-with-WiFi-Alexa/dp/B00X4WHP5E), [Amazon Tap](https://www.amazon.com/dp/B01BH83OOM), [Amazon Dot](https://www.amazon.com/b/?node=14047587011), and now [Pebble Core](https://blog.getpebble.com/2016/06/02/ks3u03/)



QUICK URLS and ACCOUNTS NECCESSARY: 

https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit#why-build-skills ** Reasons to build skills**
https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit # **Developer Portal**
https://developer.amazon.com/edw/home.html#/skills/list **List of your skills**
https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions **Lambda functions to support Alexa**
