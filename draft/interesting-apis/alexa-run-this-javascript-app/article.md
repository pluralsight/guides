# The current landscape

tutorials

[Alexa Skills Kit Template](https://developer.amazon.com/public/community/post/Tx3DVGG0K0TPUGQ/New-Alexa-Skills-Kit-Template:-Step-by-Step-Guide-to-Build-a-Fact-Skill)

[Setting up a skill](https://developer.amazon.com/public/community/post/TxDJWS16KUPVKO/New-Alexa-Skills-Kit-Template-Build-a-Trivia-Skill-in-under-an-Hour)

[Alexa App](https://github.com/matt-kruse/alexa-app)

[New SDK announced](https://developer.amazon.com/public/community/post/Tx213D2XQIYH864/Announcing-the-Alexa-Skills-Kit-for-Node-js)

[This is the one](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs)

[Not a deployment workflow](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/deploying-a-sample-skill-to-aws-lambda#preparing-a-nodejs-sample-to-deploy-in-lambda)

# How it all fits together

* export handler
* register handlers
* launch has no state then jump to other handlers using state
* Emit ask and tell responses (ignore cards)
* EmitWithState to jump to intent
* Use session attributes to persist state
* Yes/No/Help/Cancel/Stop...

# Development flow

# IMHO

# Going live