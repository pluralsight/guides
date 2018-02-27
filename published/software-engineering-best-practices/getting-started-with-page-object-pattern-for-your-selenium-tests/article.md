As with most software patterns, they really make the most sense once you have actually coded your way
into a situation where you realize you need help. Learning a pattern up front can prevent you from making a design mistake,
but as in most situations, making a mistake is your very best opportunity to learn.

The Page Object Pattern, however, makes perfect sense whether you have an existing suite of functional Selenium tests
or you are starting out fresh.

The code examples here are written in Java, but the concepts apply to any language. You can find the complete [code examples on github](https://github.com/kschiller/page-object-pattern-tutorial)

## Problem
When you are writing functional tests using Selenium a major part of your code will consist of interactions with the web interface you are testing through the WebDriver API. After fetching elements you will verify some state of the element through various assertions and move on to fetching the next element. 

Consider this example as a part of a basic Selenium test:

    List<WebElement> zipCodes = driver.findElements(By.id("zipCodes"));
    for (WebElement zipCode : zipCodes) {
        if (zipCode.getText().equals("12345")){
            zipCode.click();
            break;
        }
    }
    WebElement city = driver.findElement(By.id("city"));
    assertEquals("MyCityName", city.getText());

This is a simple example of a test that fetches and iterates over a list of zipcodes looking for 12345, clicking it and fetching the city element expecting the city name to be MyCityName.

Even with a simple test as this, readability is very poor. There is a lot of WebDriver code, that obscures the purpose of the test, making it slow and difficult to digest.

With any interface, and I suppose web interfaces in particular, it is common that both minor and major changes to the UI is implemented frequently. This could be a new design, restructuring of fields and buttons, and this will likely impact your test. So your test fails and you need to update your selectors. 

Now, if you only have one functional test for a page excersising the "happy path" you might not consider this a big deal.
However, if you have a full suite of regressions tests, this is absolutely a big deal.

So some of the typical problems for this type of Selenium test are:

*   Test cases are difficult to read
*   Changes in the UI breaks multiple tests often in several places
*   Duplication of selectors both inside and across tests - no reuse

## Solution
So instead of having each test fetch elements directly and being fragile towards UI changes, the Page Object Pattern introduces what is basically a decoupling layer.

You create an object that represents the UI you want to test, which could be a whole page or a significant part of it.
The responsibility of this object is to wrap HTML elements and encapsulate interactions with the UI, meaning that this is
where all calls to WebDriver will go. 
This is where most WebElements are. 
And this is the only place you need to modify when the UI changes.

This figure illustrates the pattern:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2a2a896f-44af-4954-820a-ba3efc485970.png)

## Implementation
Getting started with this implementation is actually very straight forward:

1. Basic Selenium setup
2. Analyze your application under test
3. Create page objects
4. Write tests

### 1. Basic Selenium setup
When you start a Selenium test suite, I recommend creating a class to hold all the driver lifecycle management code. So start out with a class like this:

    public class FunctionalTest {
        protected static WebDriver driver;

        @BeforeClass
        public static void setUp(){
            driver = new FirefoxDriver();
            driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
        }

        @After
        public void cleanUp(){
            driver.manage().deleteAllCookies();
        }

        @AfterClass
        public static void tearDown(){
            driver.close();
        }   
    }

Complete example: <https://github.com/kschiller/page-object-pattern-tutorial/blob/Initial/FunctionalTest.java>

And now make this the super class for any testclass you create.

### 2. Analyze your application under test
For this example I have created a signup form with name and address fields, and a receipt page. These examples can be seen live on [www.kimschiller.com/page-object-pattern-tutorial](http://www.kimschiller.com/page-object-pattern-tutorial).


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f487f52e-3077-4ec7-be73-4d7adb37fbf0.png)

In this example the analysis is very straight forward, and I will make a page object for each of these pages. But as mentioned above, there is not necessarily a one-to-one mapping between a complete HTML page and a page object. A page object should represent a meaningful contextual part of the page.

### 3. Create page objects
Now I will create two page objects, one for the signup form and one for the receipt.

    public class SignUpPage extends PageObject {

        @FindBy(id="firstname")
        private WebElement firstName;
        
        @FindBy(id="lastname")
        private WebElement lastName;

        @FindBy(id="address")
        private WebElement address;

        @FindBy(id="zipcode")
        private WebElement zipCode;
        
        @FindBy(id="signup")
        private WebElement submitButton;

        public SignUpPage(WebDriver driver) {
            super(driver);
        }

        public void enterName(String firstName, String lastName){
            this.firstName.clear();
            this.firstName.sendKeys(firstName);

            this.lastName.clear();
            this.lastName.sendKeys(lastName);
        }

        public void enterAddress(String address, String zipCode){
            this.address.clear();
            this.address.sendKeys(address);
            
            this.zipCode.clear();
            this.zipCode.sendKeys(zipCode);
        }
        
        public ReceiptPage submit(){
            submitButton.click();
            return new ReceiptPage(driver);
        }
    }
    
Complete example: <https://github.com/kschiller/page-object-pattern-tutorial/blob/Initial/SignUpPage.java>

    public class ReceiptPage extends PageObject {

        @FindBy(tagName = "h1")
        private WebElement header;
        
        public ReceiptPage(WebDriver driver) {
            super(driver);
        }

        public String confirmationHeader(){
            return header.getText();
        }
    }

Complete example: <https://github.com/kschiller/page-object-pattern-tutorial/blob/Initial/ReceiptPage.java>

Now, this will actually not work, until we initialize the WebElements that we have annotated. So I will make another super class to hold this small but important piece of code. So my PageObject-class looks like this:

    public class PageObject {
        protected WebDriver driver;
        
        public PageObject(WebDriver driver){
            this.driver = driver;
            PageFactory.initElements(driver, this);
        }
    }

Complete example: <https://github.com/kschiller/page-object-pattern-tutorial/blob/Initial/PageObject.java>

The PageFactory processes all the annotated WebElements and locates the element on the page using the annotated selector. You can find the documentation for PageFactory on [The SeleniumHQ GitHub page](https://github.com/SeleniumHQ/selenium/wiki/PageFactory).

### 4. Write tests
So now you have all the pieces to start writing the actual test cases.
Here is a test case going through the successful scenario of signing up and confirming:

    public class SignUpFormTest extends FunctionalTest {

        @Test
        public void signUp(){
            driver.get("http://www.kimschiller.com/page-object-pattern-tutorial/index.html");
            
            SignUpPage signUpPage = new SignUpPage(driver);
            assertTrue(signUpPage.isInitialized());

            signUpPage.enterName("First", "Last");
            signUpPage.enterAddress("123 Street", "12345");

            ReceiptPage receiptPage = signUpPage.submit();
            assertTrue(receiptPage.isInitialized());

            assertEquals("Thank you", receiptPage.confirmationHeader());
        }
    }

Complete example: <https://github.com/kschiller/page-object-pattern-tutorial/blob/Initial/SignUpFormTest.java>

Looking at the test case now, I will argue that our problem of poor readability has been resolved. The test reads nicely with the intend of each method very clear.

Since we have removed all WebDriver calls from the test, we don't really have to change the test if for instance the firstname-field changes it's id. Only the page object will need changing. 

## Best practices
There are a few best practices in using page objects, that you should make an effort to follow. 

*   A page object should not have any assertions
*   A page object should represent meaningful elements of a page and not necessarily a complete page
*   When you navigate you should return the a page object for the next page


## Tips & Tricks

### Are you ready?
It is good practice to verify that a page is ready before interacting with it, and as shown in the SignUpFormTest that is just what I am doing:

    SignUpPage signUpPage = new SignUpPage(driver);
    assertTrue(signUpPage.isInitialized());

You are likely to repeat this pattern with every instatiation of a page object,
so this is the place where we can introduce an exception to the rule-of-thumb of not having assertions in our page objects.

The idea here is to move this assertion into the constructor of the page object.
Then the creation will fail if the page object for some reason does not get ready in time.

So your constructor will look like this:

    public SignUpPage(WebDriver driver) {
        super(driver);
        assertTrue(firstName.isDisplayed());
    }

Nice and easy.

## Conclusion
 
So whether you are just getting started with Selenium or you are managing a large suite of Selenium regression tests, you will most likely benefit from introducing the Page Object Pattern into you code. As creator of WebDriver, Simon Stewart - [@shs96c](https://twitter.com/shs96c) so bluntly puts it: "If you have WebDriver APIs in your test methods, You're Doing It Wrong". And personally I have been doing it wrong, and also experienced the frustrations of doing this.

As we have now seen, the Page Object Pattern gives you a way to decouple you test scripts from the web interface you are testing, by introducing a series of page objects. And page objects are responsible for communicating with the web pages you are testing.
Any DOM queries fired through the WebDriver API, go through the page objects, because only the page objects should know how to find elements on the page using the numereous locator methods availble with Selenium.

By using page objects your tests become more concise and readable. Your element locators are centralized making maintenance easier. Changes in user interface only affects page objects and not test scripts. And finally, this is good and solid object oriented design.




