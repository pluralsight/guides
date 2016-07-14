# Using a simple wait in Selenium

In your Selenium Framework you might already have created some methods to allow you to wait for elements to be present before continuing forward. What this allows you to do is not use SLEEPS in your test code.

> **The below code is not good for tests, it's static.**

```
Thread.sleep(1000);
```

> **The code down here is dynamic and allows tests to move quickly. **

```
WebDriverWait wait = new WebDriverWait(webDriver, timeoutInSeconds);
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id<locator>));
```

Here we create an instance of the *WebDriverWait* class. The *wait* class takes 2 parameters; **webDriver** and **timoutInSeconds**. Once you have created this instance it gives you access to different methods to wait.

Here are a few of the different methods you can use to wait.

* alertIsPresent
* elementIfVisible
* elementSelectionStateToBe
* frameToBeAvailableAndSwitchToIt
* invisibilityOfElementLocated
* numberOfWindowsToBe
* presenceOfElementLocated
* textToBePresentInElement
* titleContains
* titleIs
* urlContains
* urlMatches
* visibilityOfElementLocated
* visibilityOfAllElemenentsLocated

Some other tips on using a wait is that you can create your elements with a wait. 

Simple example:
```
WebDriver wait = new WebDriverWait(webDriver, timeoutInSeconds);
WebElement button = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id<locator>));
```
A better example using the Page Object Pattern and Page Factory setup.

Your creation of the wait instance should be on the BasePage:
```
private static final String ERROR_DIV_ID = <locator>;

@FindBy(id = <locator>)
private WebElement errorDiv;

public TestingContext getTestingContext() {
    return testingContext;
}

public WebElement getErrorDiv() {
    return wait.until(ExpectedConditions.visibilityOfElementLocated(By.id(ERROR_DIV_ID)));
}

```