In this tutorial, we're going to build a chat using Pusher's presence channels with the functionality to request e-signed Non-Disclosure Agreements (NDAs) to its members using the HelloSign API.

The stack will be the following:
- Java 7 or higher
- Maven as the build manager
- Spring Boot as the server-side framework
- H2 as in-memory database
- Thymeleaf as the server-side template engine
- jQuery and Handlebars for the client-side interaction

We're going to use Pusher and HelloSign webhooks to receive events from these APIs and we'll use ngrok to keep everything in a local environment.

The app's design is based on this [pen](http://codepen.io/drehimself/pen/KdXwxR) and it works in the following way. First, a user creates a chat:

![Create chat](https://raw.githubusercontent.com/pluralsight/guides/master/images/a4a6062a-a807-42c4-9720-afdff82beb11.gif)


Then, another user joins it:

![Join chat](https://raw.githubusercontent.com/pluralsight/guides/master/images/5d064ba2-a70a-4199-bad6-2a1b3c8701f5.gif)


At any time, the chat owner can send a request to sign an NDA:

![Sending NDA](https://raw.githubusercontent.com/pluralsight/guides/master/images/401eb9e9-e6aa-4468-9446-230a9724454e.gif)


When a member of the chat signs the NDA, a notification is sent with a link to view the signed document:

![Notification signed document](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf249643-5131-47b5-b8fb-09720271e6ad.gif)



The entire source code of the final app is on [Github](https://github.com/).


# Requirements

### Java environment

You'll need JKD 7 at least, however, [JDK 8 is preferred](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).

You'll also need [Maven 3.0](https://maven.apache.org/download.cgi) or higher [installed](https://maven.apache.org/install.html).

An IDE, like [Eclipse](https://eclipse.org/downloads/), [IntelliJ](https://www.jetbrains.com/idea/) or [Netbeans](https://netbeans.org/downloads/), will make things easier but it's not required.


### Pusher

Create a free account at https://dashboard.pusher.com/accounts/sign_up.

When you first log in, you'll be asked for some configuration options:

![Pusher Account](https://raw.githubusercontent.com/pluralsight/guides/master/images/528b9445-05ee-456d-bf36-ca8c83c95bd9.png)


Enter a name, and choose *Javascript* as your front-end tech and *Java* as your back-end tech.

Then go to either the *Getting Started* or *App Keys* tab to copy your App ID, Key, and Secret credentials, we'll need them later.


### HelloSign

Sign up at https://www.hellosign.com/. Your free account will have the following limitations:
- Send 3 documents every month for free
- One sender
- No templates

But don't worry, these limitations don't apply in test mode, the mode where we're going to work.

### Ngrok
When a member is added to the chat or an NDA is signed, a webhook will be triggered (think of it as a callback). This means an HTTP request will be made to our server, so we'll need to deploy our application on the cloud or keep it locally and use a service like [ngrok](https://ngrok.com/) to make it publicly available.

Ngrok proxies external requests to your local machine by creating a secure tunnel and giving you a public URL.

ngrok is a Go program, distributed as a single executable file (no additional dependencies). So for now, just download it from [https://ngrok.com/download](https://ngrok.com/download) and unzip the compressed file.

Now that we have all we need, let's create the app.


# Setting up the application

One of the easiest ways to create a Spring Boot app is to use the project generator at [https://start.spring.io/](https://start.spring.io/).

Go to that page and choose to generate a Maven project with the following dependencies:
- Web
- H2
- Thymeleaf
- JPA

Enter a Group ID and an Artifact ID and generate the project:

![Sprint Initializr](https://raw.githubusercontent.com/pluralsight/guides/master/images/17e7968c-3234-4f8f-822d-ef1e34906060.png)


Unzip the content of the downloaded file. At this point, you can import the project to an IDE if you want. For example, in Eclipse, go to *File -> Import* and choose *Existing Maven Projects*:

![Import Maven Project into Eclipse](https://raw.githubusercontent.com/pluralsight/guides/master/images/83970ac2-1e10-4e8d-b5cf-57c53ed8c423.png)

Now let's add some configurations to the `pom.xml` file. In the `properties` section, change the Java version if you're not using Java 8 and add the following line:
```xml
<spring.version>4.3.1.RELEASE</spring.version>
```

The latest version of the Spring Framework at the time of this writing is `4.3.1.RELEASE`. This line will ensure Spring Boot uses this version.

Also, in the `dependencies` section, add the dependencies we'll need for our project:
```xml
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-lang3</artifactId>
	<version>3.4</version>
</dependency>

<dependency>
  <groupId>com.pusher</groupId>
  <artifactId>pusher-http-java</artifactId>
  <version>1.0.0</version>
</dependency>

<dependency>
	<groupId>com.hellosign</groupId>
	<artifactId>hellosign-java-sdk</artifactId>
	<version>3.4.0</version>
</dependency>
```

Inside `src/main/java`, we'll work with the following package structure:
- `com.example.config` will contain configuration classes
- `com.example.constants` will contain interfaces with constants values used in the app
- `com.example.model` will contain the JPA entity models
- `com.example.repository` will contain the Spring JPA interfaces to work with the model
- `com.example.service` will contain the business service classes of the app
- `com.example.web` will contain the Spring MVC controllers of the app
- `com.example.web.vo` will contain the objects used for communication between the view and the controllers

Inside `src/main/resources`, we'll put some configuration files in addition to the following directory structure:
- `static/css` will contain the CSS style files used in the application
- `static/img` will contain the images used in the application
- `static/js` will contain the Javascript files used in the application
- `templates` will contain the Thymeleaf templates used in the application

So let's start by creating the `com/example/web/ChatController` class. Create a new file with the following content:
```java
@Controller
public class ChatController {
	
	@RequestMapping(method=RequestMethod.GET, value="/")
    public ModelAndView index() {
		ModelAndView modelAndView = new ModelAndView();
		
		modelAndView.setViewName("index");
		modelAndView.addObject("text", "Hello World!"); 
        
        return modelAndView;
    }
}
```

This controller defines a `/` route that shows a `index` template. Create the file `src/main/resources/templates/index.html` with some HTML content like:
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>NDA Chat</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="${text}" />
</body>
</html>
```

Using Thymeleaf syntax, this will print the `text` variable defined in the controller. You can learn more about [Thymeleaf here](http://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html).

Run the application either by executing the `com.example.NdaChatApplication` class on your IDE or on the command line with:
```
$ mvn spring-boot:run
```

Additionally, on the command line, you can create a JAR file to execute it:
```
$ mvn package -DskipTests
$ java -jar target/nda-chat-0.0.1-SNAPSHOT.jar
```

When you open `http://localhost:8080/` in a browser, you should see something like the following:

![Hello World on localhost](https://raw.githubusercontent.com/pluralsight/guides/master/images/ae127e34-6b69-43f4-b34a-cd190edd8a6d.png)


Now, let's take a moment to configure ngrok.

# Setting up Ngrok
In a new terminal window, navigate to the directory where you unzipped ngrok. 

We'll start ngrok by telling it which port we want to expose to the Internet. For our app, that would be port `8080`:
```
./ngrok http 8080
```

Alternatively, if you're on Windows:
```
ngrok http 8080
```

Now, you should see something like this:

![Ngrok console](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd9d375c-59cb-4450-abf3-aac611fe2fe9.png)


See that URL in the *Forwarding* row(s) with the `ngrok.io` domain? That's your public URL. Yours will be different, ngrok generates a random URL every time you run it.

If you open in a browser `http://[YOUR_GENERATED_SUBDOMAIN].ngrok.io`, you should see the same page found on `http://localhost:8080`:

![Hello World with Ngrok](https://raw.githubusercontent.com/pluralsight/guides/master/images/5e573f70-4df4-44a8-b3e9-a65dc3fa6097.png)



Open this ngrok URL in another computer if you want. Once again, you should see the same page. Our local server is now available publicly.

The only disadvantage is that this URL is not permanent. If you restart ngrok, it will give you another URL.

You can specify a subdomain, for example, to get the URL  `http://chat.ngrok.io` use the command:
```
ngrok http -subdomain=chat 8080
```

However, this requires a paid plan. You can get more info in this [page](https://ngrok.com/product).

Nevertheless, as long as you don't stop or restart ngrok, the URL won't change, so forget about that terminal window and let's leave it running for now.

In the next section, we'll deep into the code of the application, starting with the database layer.


# Database layer
We'll use [H2](http://www.h2database.com/html/main.html) as the database for our application. It can work as an embedded in-memory database, which is perfect for our purposes.

Open the `src/main/resources/application.properties` file and specify the following options:
```
spring.datasource.platform=h2

spring.jpa.hibernate.ddl-auto="none"

spring.h2.console.enabled=true

spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.type=trace
```

In Spring Boot, by default, JPA databases will be created automatically if you use an embedded database (such as H2, HSQL or Derby) by executing the files `schema.sql` (to define the structure of the database) and `data.sql` (to insert initial data) in the classpath.

However, we can define *profiles* to run distinct scripts for different databases. That's the purpose of the line:
```
spring.datasource.platform=h2
```

It tells Spring Boot to execute the files `schema-h2.sql` and `data-h2.sql`. For this app, create the file `src/main/resources/schema-h2.sql` with the following content:
```sql
CREATE TABLE chat(
	chat_id 			BIGINT IDENTITY PRIMARY KEY,
	chat_name 			VARCHAR(50) NOT NULL,
	chat_description 	VARCHAR(150) NOT NULL,
	chat_active			BOOLEAN NOT NULL,
	created_at			TIMESTAMP NOT NULL
);

CREATE TABLE user(
	user_id 			BIGINT IDENTITY PRIMARY KEY,
	user_email 			VARCHAR(50) NOT NULL,
	user_name			VARCHAR(50) NOT NULL,
	user_active			BOOLEAN NOT NULL,
	nda_signed			BOOLEAN NOT NULL,
	owns_chat			BOOLEAN NOT NULL,
	chat_id				BIGINT NOT NULL,
	sign_id 			VARCHAR(255),
	created_at			TIMESTAMP NOT NULL,
	CONSTRAINT FK_user_chat FOREIGN KEY (chat_id) REFERENCES chat(chat_id)
);

CREATE TABLE message(
	message_id			BIGINT IDENTITY PRIMARY KEY,
	chat_id				BIGINT NOT NULL,
	user_id				BIGINT NOT NULL,
	message_text	 	VARCHAR(1000) NOT NULL,
	created_at			TIMESTAMP NOT NULL,
	CONSTRAINT FK_message_chat FOREIGN KEY (chat_id) REFERENCES chat(chat_id),
	CONSTRAINT FK_message_user FOREIGN KEY (user_id) REFERENCES user(user_id)
);
```

By default, Hibernate (the JPA implementation used by Spring Boot), will try to create the schema. Since Spring Boot will be responsible for that, we have to turn off this feature with:
```
spring.jpa.hibernate.ddl-auto="none"
```

H2 provides a [browser-based console](http://www.h2database.com/html/quickstart.html#h2_console) that Spring Boot can auto-configure at the path `/h2-console` for you when these conditions are met:
- You are developing a web application
- `com.h2database:h2` is on the classpath
- You are using [Spring Boot's developer tools](http://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)

Since we're not using Spring Boot's developer tools, we have to explicitly configure it with:
```
spring.h2.console.enabled=true 
```

The rest of the properties in the `application.properties` file tell Hibernate to print in the console the generated SQL statements for debugging purposes.

Now that we have our schema, let's create the JPA entities that will represent the database tables.

Create the `src/main/java/com/example/model/Chat.java` file with the following content:
```java
package com.example.model;

import java.io.Serializable;
import java.util.Date;
import java.util.LinkedHashSet;
import java.util.Set;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;
import org.apache.commons.lang3.builder.ToStringBuilder;

/**
 * Object that represents a Chat entity
 */
@Entity
public class Chat  implements Serializable {
	/**
	 * UID for serialization
	 */
	private static final long serialVersionUID = 1598739731826887591L;

	/** ID of the chat */
	@Id
	@Column(name="chat_id")
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	
	/** Name of the chat */
	@Column(name="chat_name", nullable = false)
	private String name;
	
	/** Description of the chat */
	@Column(name="chat_description", nullable = false)
	private String description;
	
	/** Indicates if the chat is active */
	@Column(name="chat_active", nullable = false)
	private Boolean active;
	
	/** The date and time when the chat was created */
	@Column(name="created_at", nullable = false)
	private Date createdAt;
	
	/** Members of the chat */
	@OneToMany(fetch = FetchType.LAZY, 
            cascade = CascadeType.ALL, 
            mappedBy = "chat", 
            orphanRemoval = true)
	private Set<User> members = new LinkedHashSet<User>();
	
	/**
	 * Helper method to add a member
	 * @param user Member to add to the chat
	 */
	public void addMember(User user) {
        this.members.add(user);
        user.setChat(this);
    }

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDescription() {
		return description;
	}

	public void setDescription(String description) {
		this.description = description;
	}

	public Boolean getActive() {
		return active;
	}

	public void setActive(Boolean active) {
		this.active = active;
	}

	public Date getCreatedAt() {
		return createdAt;
	}

	public void setCreatedAt(Date createdAt) {
		this.createdAt = createdAt;
	}

	public Set<User> getMembers() {
		return members;
	}

	public void setMembers(Set<User> members) {
		this.members = members;
	}

	@Override
	public String toString(){
		return new ToStringBuilder(this).
			       append("id", id).
			       append("name", name).
			       append("description", description).
			       append("active", active).
			       append("createdAt", createdAt).
			       toString();
	}
	
	@Override
	public boolean equals(Object obj) {
	   if (obj == null) { return false; }
	   if (obj == this) { return true; }
	   if (obj.getClass() != getClass()) {
	     return false;
	   }
	   Chat that = (Chat) obj;
	   return new EqualsBuilder()
	                 .appendSuper(super.equals(obj))
	                 .append(id, that.id)
	                 .isEquals();
	  }
	
	@Override
	public int hashCode() {
		return new HashCodeBuilder(5, 13).
		   append(id).
		   toHashCode();
	}
}
```

Notice how the one-to-many relationship with the `User` entity is set up and the helper method to establish it. In addition, as the good practices dictate, we're also defining  the `toString()`, `equals()`, and `hashCode()` methods with the help classes from the [commons-lang](https://commons.apache.org/proper/commons-lang/) library.

The `src/main/java/com/example/model/User.java` file contains the following code:
```java
package com.example.model;

import java.io.Serializable;
import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;
import org.apache.commons.lang3.builder.ToStringBuilder;

/**
 * Object that represents a User entity
 */
@Entity
public class User implements Serializable {
	/**
	 * UID for serialization
	 */
	private static final long serialVersionUID = 3868269731826822792L;

	/** ID of the user */
	@Id
	@Column(name="user_id")
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	
	/** Email of the user */
	@Column(name="user_email", nullable = false)
	private String email;
	
	/** Name of the user */
	@Column(name="user_name", nullable = false)
	private String name;
	
	/** Indicates if the user is active */
	@Column(name="user_active", nullable = false)
	private Boolean active;
	
	/** Indicates if the user has signed the NDA */
	@Column(name="nda_signed", nullable = false)
	private Boolean ndaSigned;
	
	/** Indicates if the user is the owner of the chat */
	@Column(name="owns_chat", nullable = false)
	private Boolean ownsChat;
	
	/** ID of the HelloSign request to sign the NDA */
	@Column(name="sign_id", nullable = true)
	private String signId;
	
	/** Date and time of the creation of the user */
	@Column(name="created_at", nullable = false)
	private Date createdAt;
	
	/** Object that represents the chat the user belongs to */
	@ManyToOne
    @JoinColumn(name = "chat_id", 
                nullable = false, updatable = false)
	private Chat chat;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Boolean getActive() {
		return active;
	}

	public void setActive(Boolean active) {
		this.active = active;
	}

	public Boolean getNdaSigned() {
		return ndaSigned;
	}

	public void setNdaSigned(Boolean ndaSigned) {
		this.ndaSigned = ndaSigned;
	}

	public Boolean getOwnsChat() {
		return ownsChat;
	}

	public void setOwnsChat(Boolean ownsChat) {
		this.ownsChat = ownsChat;
	}

	public String getSignId() {
		return signId;
	}

	public void setSignId(String signId) {
		this.signId = signId;
	}

	public Date getCreatedAt() {
		return createdAt;
	}

	public void setCreatedAt(Date createdAt) {
		this.createdAt = createdAt;
	}

	public Chat getChat() {
		return chat;
	}

	public void setChat(Chat chat) {
		this.chat = chat;
	}

	@Override
	public String toString(){
		return new ToStringBuilder(this).
			       append("id", id).
			       append("email", email).
			       append("name", name).
			       append("active", active).
			       append("ndaSigned", ndaSigned).
			       append("ownsChat", ownsChat).
			       append("createdAt", createdAt).
			       toString();
	}
	
	@Override
	public boolean equals(Object obj) {
	   if (obj == null) { return false; }
	   if (obj == this) { return true; }
	   if (obj.getClass() != getClass()) {
	     return false;
	   }
	   User that = (User) obj;
	   return new EqualsBuilder()
	                 .appendSuper(super.equals(obj))
	                 .append(id, that.id)
	                 .isEquals();
	  }
	
	@Override
	public int hashCode() {
		return new HashCodeBuilder(3, 11).
		   append(id).
		   toHashCode();
	}
}
```

In turn, `src/main/java/com/example/model/Message.java` contains:
```java
package com.example.model;

import java.io.Serializable;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;
import org.apache.commons.lang3.builder.ToStringBuilder;

/**
 * Object that represents a Message entity
 */
@Entity
public class Message implements Serializable {

	/**
	 * UID for serialization
	 */
	private static final long serialVersionUID = 7024644638730524732L;

	/** ID of the message */
	@Id
	@Column(name="message_id")
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	
	/** ID of the chat that the message belongs to */
	@Column(name="chat_id", nullable = false)
	private Long idChat;
	
	/** Text of the message */
	@Column(name="message_text", nullable = false)
	private String message;
	
	/** Date and time when the message was created */
	@Column(name="created_at", nullable = false)
	private Date createdAt;
	
	/** Object that represents the user that post the message */
	@ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "user_id", 
                nullable = false, updatable = false)
	private User user;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public Long getIdChat() {
		return idChat;
	}

	public void setIdChat(Long idChat) {
		this.idChat = idChat;
	}

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}

	public Date getCreatedAt() {
		return createdAt;
	}

	public void setCreatedAt(Date createdAt) {
		this.createdAt = createdAt;
	}

	public User getUser() {
		return user;
	}

	public void setUser(User user) {
		this.user = user;
	}
	
	public String getCreatedAtString() {
		String time = "";
		if(this.createdAt != null) {
			DateFormat df = new SimpleDateFormat("hh:mm a");
			time = df.format(this.createdAt);
		}
		
		return time;
	}
	
	public String getMessageFormatted() {
		String msg = "";
		
		if(this.message != null) {
			msg = this.message.replaceAll("(\r?\n)", "<br />");
		}
		
		return msg;
	}

	@Override
	public String toString(){
		return new ToStringBuilder(this).
			       append("id", id).
			       append("idChat", idChat).
			       append("message", message).
			       append("createdAt", createdAt).
			       toString();
	}
	
	@Override
	public boolean equals(Object obj) {
	   if (obj == null) { return false; }
	   if (obj == this) { return true; }
	   if (obj.getClass() != getClass()) {
	     return false;
	   }
	   Message that = (Message) obj;
	   return new EqualsBuilder()
	                 .appendSuper(super.equals(obj))
	                 .append(id, that.id)
	                 .isEquals();
	  }
	
	@Override
	public int hashCode() {
		return new HashCodeBuilder(7, 15).
		   append(id).
		   toHashCode();
	}
}
```

The repository objects will be handled by [Spring Data JPA](http://projects.spring.io/spring-data-jpa/), which reduces the amount of boilerplate code by generating the code to implement the data-access layer code from interfaces you define. You can read about how to work with [Spring Data in this page](http://docs.spring.io/spring-data/jpa/docs/current/reference/html/).

By default, Spring Data creates the following methods for basic [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) functionality:
```java
<S extends T> S save(S entity); 
T findOne(ID primaryKey);       
Iterable<T> findAll();          
Long count();                   
void delete(T entity);          
boolean exists(ID primaryKey);
```

This way, we just have to define the business methods used by the application.

For the Chat repository, those would be:
```java
package com.example.repository;

import java.util.List;

import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.example.model.Chat;

/**
 * Repository that contains operations to work with a Chat entities
 */
@Repository
public interface ChatRepository extends CrudRepository<Chat, Long> {

	/**
	 * Finds all active chats
	 * @return a List of Chat objects ordered by their ID
	 */
	List<Chat> findByActiveTrueOrderById();

	/**
	 * Finds an active chat by its name
	 * @param chatName Name of the chat
	 * @return List of Chat objects that meet the search criteria
	 */
	List<Chat> findByNameAndActiveTrue(String chatName);
}
```

The User repository:
```java
package com.example.repository;

import java.util.List;

import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.example.model.Chat;
import com.example.model.User;

/**
 * Repository that contains operations to work with User entities
 */
@Repository
public interface UserRepository extends CrudRepository<User, Long> {

	/**
	 * Finds active members of a chat who haven't sign the NDA and are not the chat owner
	 * @param chat the Chat object that the users belongs to
	 * @return A List of User objects that represent the members of the chat
	 */
	List<User> findByChatAndNdaSignedFalseAndActiveTrueAndOwnsChatFalse(Chat chat);
	
	/**
	 * Finds users by their sign request ID
	 * @param signId HelloSign sign request ID
	 * @return A List of User objects that meet the search criteria
	 */
	@EntityGraph(attributePaths = { "chat" })
	List<User> findBySignId(String signId);
}
```

The `@EntityGraph` annotation indicates that the chat relationship object must be fetched.

The Message repository:
```java
package com.example.repository;

import java.util.List;

import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.example.model.Message;

/**
 * Repository that contains operations to work with a Message entities
 */
@Repository
public interface MessageRepository extends CrudRepository<Message, Long> {

	/**
	 * Finds the messages of a chat
	 * @param idChat ID of the chat
	 * @return a List of Message objects that meet the search criteria
	 */
	List<Message> findByIdChatOrderByCreatedAt(Long idChat);
}
```

Next, we're going to review the service layer.

# Service layer

The service layer is just a thin wrapper of the database layer. Its main purpose is to provide transactional capabilities to data access methods.

We're going to set up two services. The chat service:
```java
package com.example.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.model.Chat;
import com.example.model.Message;
import com.example.model.User;
import com.example.repository.ChatRepository;
import com.example.repository.MessageRepository;
import com.example.repository.UserRepository;

/**
 * Service with the operations to work with a chat
 */
@Service
@Transactional
public class ChatService {

	@Autowired
	private UserRepository userRepository;

	@Autowired
	private ChatRepository chatRepository;

	@Autowired
	private MessageRepository messageRepository;
	
	/**
	 * Saves(creates or modify) a chat object
	 * @param chat The object to save
	 */
	public void saveChat(Chat chat) {
		chatRepository.save(chat);
	}
	
	/**
	 * Add a new user to a chat
	 * @param id ID of the chat
	 * @param user Object that represents the user to add
	 * @return the Chat object with the new user added
	 */
	public Chat addNewUserToChat(Long id, User user) {
		Chat chat = getChat(id);
		chat.addMember(user);
		
		userRepository.save(user);
		
		return chat;
	}
	
	/**
	 * Saves a message and its relation to a user
	 * @param msg The message to save
	 * @param idUser The ID of the user that post the message
	 */ 
	public void saveMessage(Message msg, Long idUser) {
		User user = userRepository.findOne(idUser);
		msg.setUser(user);
		
		messageRepository.save(msg);
	}
	
	/**
	 * Marks an active chat as inactive
	 * @param chatName The name of the chat to mark
	 */
	public void markChatAsInactive(String chatName) {
		List<Chat> chats = chatRepository.findByNameAndActiveTrue(chatName);
		
		if(chats != null && !chats.isEmpty()) {
			Chat chat = chats.get(0);
			chat.setActive(Boolean.FALSE);
			chatRepository.save(chat);
		}
	}
	
	/**
	 * Finds a chat by its ID
	 * @param id ID of the chat to find
	 * @return a chat object
	 */
	public Chat getChat(Long id) {
		return chatRepository.findOne(id);
	}
	
	/**
	 * Get all the active chats
	 * @return a List of chat objects
	 */
	public List<Chat> getAllActiveChats() {
		return chatRepository.findByActiveTrueOrderById();
	}
	
	/**
	 * Find an active chat by name
	 * @param chatName Name of the chat
	 * @return a chat object
	 */
	public Chat getActiveChatByName(String chatName) {
		List<Chat> chats = chatRepository.findByNameAndActiveTrue(chatName);
		Chat chat = null;
		
		if(chats != null && !chats.isEmpty()) {
			chat = chats.get(0);
		}
		
		return chat;
	}
	
	/**
	 * Gets all the messages of a chat
	 * @param idChat ID of the chat
	 * @return a List of message objects
	 */
	public List<Message> getAllChatMessages(Long idChat) {
		return messageRepository.findByIdChatOrderByCreatedAt(idChat);
	}
}
```

And the user service:
```java
package com.example.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.model.Chat;
import com.example.model.User;
import com.example.repository.ChatRepository;
import com.example.repository.UserRepository;

/**
 * Service with the operations to work with a user
 */
@Service
@Transactional
public class UserService {

	@Autowired
	private UserRepository userRepository;

	@Autowired
	private ChatRepository chatRepository;
	
	/**
	 * Marks a user as inactive
	 * @param idUser ID of the user
	 */
	public void markUserAsInactive(Long idUser) {
		User user = userRepository.findOne(idUser);
		
		if(user != null) {
			user.setActive(Boolean.FALSE);
			userRepository.save(user);
		}
	}

	/**
	 * Find all the active members of a chat that haven't sign the NDA
	 * @param idChat ID of the chat
	 * @return a List of User objects
	 */
	public List<User> getChatMembersToSignNda(Long idChat) {
		Chat chat = chatRepository.findOne(idChat);
		
		return userRepository.findByChatAndNdaSignedFalseAndActiveTrueAndOwnsChatFalse(chat);
	}
	
	/**
	 * Get a user by the HelloSign ID of her signature request
	 * @param signId ID of the signature request
	 * @return a User object
	 */
	public User getUserBySignId(String signId) {
		List<User> users = userRepository.findBySignId(signId);
		User user = null;
		
		if(users != null && !users.isEmpty()) {
			user = users.get(0);
		}
		
		return user;
	}
	
	/**
	 * Saves(creates or modify) a user object
	 * @param user The object to save
	 */
	public void saveUser(User user) {
		userRepository.save(user);
	}
}
```

Finally, in the `src/main/java/com/example/constants/GeneralConstants.java` and `src/main/java/com/example/constants/HelloSignConstants.java` files, we'll define the constants we'll use in the next sections:
```java
package com.example.constants;

/**
 * General application constants
 */
public interface GeneralConstants {

	/** Channel prefix required by Pusher for presence chats */
	String CHANNEL_PREFIX = "presence-";
	
	/** ID used to store the object in the session */
	String ID_SESSION_CHAT_INFO = "chatInfo";
}
```

```java
package com.example.constants;

/**
 * Constants used for the HelloSign API
 */
public interface HelloSignConstants {

	/** Subject for the email sent by HelloSign to request a sign */
	String EMAIL_SUBJECT = "The NDA for the chat ";
	
	/** ID of the signing role */
	String SIGNING_ROLE = "Consultant";
	
	/** ID of the template custom field */
	String NAME_TEMPLATE_FIELD = "name";
	
	/** Error message when the file of the signed document doesn't exist */
	String FILE_DOWNLOAD_ERROR_MSG = "Sorry. The file does not exist.";
	
	/** Content type of the file */
	String FILE_CONTENT_TYPE = "application/pdf";
	
	/** ID for the request signed event */
	String REQUEST_SIGNED_EVENT = "signature_request_signed";
	
	/** ID for the request sent event */
	String REQUEST_SENT_EVENT = "signature_request_sent";
	
	/** Response to the webhook request required by HelloSign */
	String WEBHOOK_RESPONSE = "Hello API Event Received";
}
```

Now that we've taken care of most of the boilerplate code, let's go where the fun part resides, the controller and the view layers.


# Setting up the initial page

Open the `ChatController` class and modify the `index` method so it looks like this:
```java
public class ChatController {
	
	private Logger logger = LoggerFactory.getLogger(ChatController.class);
	
	@Autowired
	private ChatService chatService;
	
	/**
	 * Route for the main page
	 * @return Object with the view information
	 */
	@RequestMapping(method=RequestMethod.GET, value="/")
    public ModelAndView index() {
		ModelAndView modelAndView = new ModelAndView();
		
		List<Chat> list = chatService.getAllActiveChats();
		logger.debug("" + list.size());
		
		modelAndView.setViewName("index");
		modelAndView.addObject("chat", new ChatForm()); 
		modelAndView.addObject("chats", list);
        
        return modelAndView;
    }
}
```

This will fetch all the active chats to present them on the index page. Next, modify the `index.html` template:
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>NDA Chat</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.2.0/animate.min.css" />
    <link rel="stylesheet" href="/css/style.css" />
</head>
<body>
	<div class="container clearfix"> 
		<div class="options-container">
		    <a id="createChat" href="#createChatModal">Create a new chat</a>
		    
		    <div th:unless="${#lists.isEmpty(chats)}">
		    OR
		    
		    <a id="joinChat" href="#joinChatModal">Join an existing chat</a>
		    </div>
    	</div>
    </div>

    <div id="createChatModal">
        <div id="close-container" class="close-createChatModal"> 
            <img class="close-icon" src="img/close.svg" />
        </div>
            
        <div class="modal-content">
        	<form id="create-chat-form" role="form" th:action="@{/chat/create}" method="post" th:object="${chat}">
			    <div>
			        <label for="chatName">Chat Name</label>
			        <input type="text" id="chatName" name="chatName" maxlength="50" size="35" th:field="${chat.chatName}"/>
			    </div>
			    <div>
			        <label for="chatDescription">Chat Description</label>
			        <input type="text" id="chatDescription" name="chatDescription" maxlength="150" size="60" th:field="${chat.chatDescription}"/>
			    </div>
			    <div>
			        <label for="userEmail">Your E-mail:</label>
			        <input type="text" id="userEmail" name="userEmail" maxlength="50" size="35" th:field="${chat.userEmail}"/>
			    </div>
			    <div>
			        <label for="userName">Your Name:</label>
			        <input type="text" id="userName" name="userName" maxlength="50" size="35" th:field="${chat.userName}"/>
			    </div>
			    <div>
			        <button type="submit">Save</button>
			    </div>
			</form>
        </div>
    </div>

    <div id="joinChatModal" th:unless="${#lists.isEmpty(chats)}">
        <div id="close-container" class="close-joinChatModal"> 
            <img class="close-icon" src="img/close.svg" />
        </div>
            
        <div class="modal-content">
        	<form id="join-chat-form" role="form" th:action="@{/chat/join}" method="post" th:object="${chat}">
			    <div>
			        <label for="idChat">Chat</label>
			        <select id="idChat" name="idChat" th:field="${chat.idChat}">
			        	<option value="">---- Select ----</option>
			        	<option	  th:each="chat : ${chats}" 
						          th:value="${chat.id}" 
						          th:text="${chat.name}"></option>
			        </select>
			    </div>
			    <div>
			        <label for="userEmail">Your E-mail:</label>
			        <input type="text" id="userEmail" name="userEmail" maxlength="50" size="35" th:field="${chat.userEmail}"/>
			    </div>
			    <div>
			        <label for="userName">Your Name:</label>
			        <input type="text" id="userName" name="userName" maxlength="50" size="35" th:field="${chat.userName}"/>
			    </div>
			    <div>
			        <button type="submit">Save</button>
			    </div>
			</form>
        </div>
    </div>
    
    <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
    <script src="http://ajax.aspnetcdn.com/ajax/jquery.validate/1.15.0/jquery.validate.min.js"></script>
	<script src="js/animatedModal.min.js"></script>
	<script src="js/validate.js"></script>
	
	<script>
		$("#createChat").animatedModal({
			modalTarget:'createChatModal',
			color: '#94C2ED'
		});
	</script>
	
	<script th:unless="${#lists.isEmpty(chats)}">
		$("#joinChat").animatedModal({
	        modalTarget:'joinChatModal',
			color: '#94C2ED'
		});
	</script>
</body>
</html>
```

Some remarks on this page:
- You can find the code of the CSS and JS files in the [Github repo](https://github.com).
- If the list of chats is empty, the options related to joining an existing chat are not rendered.
- It uses [animatedModal.js](http://joaopereirawd.github.io/animatedModal.js/) to present the forms to create or join a chat in a modal window.
- The forms are validated with the [jQuery Validation Plugin](https://jqueryvalidation.org/). The validation rules can be found on the `validate.js` script.
- When creating a chat, the name must be unique among the active chats. This validation is made with an AJAX call to the following method on the chat controller:
```java
/**
 * Route to validate that a chat name is unique
 * @param chatName The name to validate
 * @return true if the name is valid (doesn't exist) or false otherwise
 */
@RequestMapping(method=RequestMethod.GET, value="/chat/validate/name", produces = "application/json")
@ResponseBody
public String validateChatName(@RequestParam String chatName) {
	Chat chat = chatService.getActiveChatByName(chatName);
	
	return String.valueOf(chat == null);
}
```

When a chat is created, this method on the controller is invoked:
```java
/**
 * Route to create a chat
 * @param form Object with the chat information
 * @param model Object to store information in the session
 * @return the view to redirect the request
 */
@RequestMapping(method=RequestMethod.POST, value="/chat/create")
public String createChat(ChatForm form, Model model) {
	
	User user = new User();
	user.setName(form.getUserName());
	user.setActive(Boolean.TRUE);
	user.setCreatedAt(new Date());
	user.setEmail(form.getUserEmail());
	user.setNdaSigned(Boolean.FALSE);
	user.setOwnsChat(Boolean.TRUE);
	
	Chat chat = new Chat();
	chat.setActive(Boolean.TRUE);
	chat.setCreatedAt(new Date());
	chat.setDescription(form.getChatDescription());
	
	form.setChatName(form.getChatName().toLowerCase().replaceAll("\\s+", "-")); // Replace blank spaces with a hyphen
	form.setPresenceChatName(GeneralConstants.CHANNEL_PREFIX + form.getChatName());
	chat.setName(form.getChatName());
	
	chat.addMember(user);
	
	chatService.saveChat(chat);
	
	form.setIsUserChatOwner(Boolean.TRUE);
	form.setIdChat(chat.getId());
	form.setIdUser(user.getId());
	
	model.addAttribute(GeneralConstants.ID_SESSION_CHAT_INFO, form);
	
	return "redirect:/chat";
}
```

Notice how the helper method `addMember()` is used to set up the relationship between the chat and the user so both entities can be saved with a single method call on the `Chat` object. Also, the channel name is prefixed with `presence-` (defined as `GeneralConstants.CHANNEL_PREFIX`), which is required by Pusher presence channels (more on this later). Finally, the `ChatForm` is added to the `Model` object.

Now, to save this object to the HTTP session, we only have to annotate the controller class with `@SessionAttributes` and the same identifier used when it was added to the `Model`  object:
```java
@Controller
@SessionAttributes(GeneralConstants.ID_SESSION_CHAT_INFO)
public class ChatController {
	...
}
```

This way, the chat information will be available to all pages of our web app. 

The method to join an existing chat is similar:
```java
**
 * Route to join an existing chat
 * @param form Object with the chat information
 * @param model Object to store information in the session
 * @return the view to redirect the request
 */
@RequestMapping(method=RequestMethod.POST, value="/chat/join")
public String joinChat(ChatForm form, Model model) {
	User user = new User();
	user.setName(form.getUserName());
	user.setActive(Boolean.TRUE);
	user.setCreatedAt(new Date());
	user.setEmail(form.getUserEmail());
	user.setNdaSigned(Boolean.FALSE);
	user.setOwnsChat(Boolean.FALSE);
	
	Chat chat = chatService.addNewUserToChat(form.getIdChat(), user);
	
	form.setIsUserChatOwner(Boolean.FALSE);
	form.setIdUser(user.getId());
	form.setChatName(chat.getName());
	form.setChatDescription(chat.getDescription());
	form.setPresenceChatName(GeneralConstants.CHANNEL_PREFIX + form.getChatName());
	
	model.addAttribute(GeneralConstants.ID_SESSION_CHAT_INFO, form);
	
	return "redirect:/chat";
}
```

As you can see, both methods redirect to the `/chat` route, which is defined by this method:
```java
/**
 * Route for the chat page
 * @param chatInfo Session Object that contains information about the chat
 * @return Object with the view information
 */
@RequestMapping(method=RequestMethod.GET, value="/chat")
public ModelAndView chat(
		@SessionAttribute(GeneralConstants.ID_SESSION_CHAT_INFO) ChatForm chatInfo) {
	ModelAndView modelAndView = new ModelAndView();
	List<Message> list = chatService.getAllChatMessages(chatInfo.getIdChat());
	
	modelAndView.setViewName("chat");
	modelAndView.addObject("messages", list);
	
	return modelAndView;
}
```

This method gets all the existing chat messages (if any) to present the chat history to the user.

Next, let's talk about Pusher presence chats, set up a Pusher webhook, and the configurations in HelloSign so a user can sign an NDA document.

# Pusher's presence chats and Webhooks