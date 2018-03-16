# Introduction
The [Representational State Transfer](https://en.wikipedia.org/wiki/Representational_state_transfer) (REST) architecture is the most popular way to expose data from a server through an [API](https://en.wikipedia.org/wiki/Application_programming_interface).

REST defines a set of principles that must be followed. There's a certain degree of freedom, but the clients have to be adapted to the way in which the resources are represented on the server side, which makes REST not a good fit for some cases.

[GraphQL](http://graphql.org/) is a query language that offers an alternative model to developing APIs using a strong type system to provide a detailed description of an API.

In this tutorial, you're going to learn how to build a GraphQL server that will expose an API to create, update and delete entities of type `Book` with the following attributes:
- ID
- Title
- ISBN
- Page count
- Author

`Author` will be a complex type with the following attributes:
- ID
- First name
- Last name

You'll define these types in a GraphQL schema using [a special Graphql DSL called IDL](http://graphql-java.readthedocs.io/en/v6/schema.html). You'll also create resolvers, queries, mutations, error handlers and the rest of the infrastructure for the server.

This tutorial will use:
- [Java 9](http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html)
- [Spring Boot 1.5.9](https://projects.spring.io/spring-boot/) with an embedded [H2 database](http://www.h2database.com/html/main.html)
- [graphql-java](https://github.com/graphql-java/graphql-java), a GraphQL Java implementation
- [GraphiQL](https://github.com/skevy/graphiql-app), an app for editing and testing GraphQL queries/mutations (you'll learn how to use its web version)
- [Maven](https://maven.apache.org/) (but you can use [Gradle](https://gradle.org/) if you prefer it)

At the end of the tutorial, you'll have a strong foundation to build APIs with GraphQL using Spring Boot.

You can find the entire source code of this project on this [GitHub repository](https://github.com/eh3rrera/graphql-java-spring-boot-example).

Let's start by talking about GraphQL.

# What is GraphQL?

GraphQL is a query language for APIs created by Facebook in 2012, open-sourced in 2015, and now released as a [specification](http://facebook.github.io/graphql/October2016/).

GraphQL describes the data offered by an API through a schema that contains:
- Data types and relationships between them
- A set of operations:
  - Queries to get data
  - Mutations to create, update, and delete data
  
For example, the following snippet defines the types `Song` and `Artist` along with their attributes and relationship between them:
```
type Song {
   id: ID!
   title: String!
   duration: Int!
   genre: String!
   artist: Artist!
}

type Artist {
   id: ID!
   name: String!
   country: String!
   songs: [Song]
}
```
Notice that the fields are typed. These types can be scalar types (`Int`, `Float`, `String`, `Boolean` and `ID`) or references to other types defined in the specification.

You can also specify if they are required (`!`) or if they are an array (`[]`). Here you can find more information about [object types and fields](http://graphql.org/learn/schema/#object-types-and-fields).

Query operations are also treated as types. They declare fields that represent the available operations. For example, you can define query operations to get all the songs and filter the songs by genre:
```
type SongQueries {
   allSongs: [Song]
   filterSongsByGenre(genre: String!): [Song]
}

schema {
   query: SongQueries
}
```

By specifying a return type, GraphQL allows you to request only the information you need from a resource. Here's a sample query to get the name of all the rock songs:
```
{
  filterSongsByGenre(genre: "rock") {
    title
  }
}
```

And even request the information of related resources:
```
{
  filterSongsByGenre(genre: "rock") {
    title
    artist {
      name
      country
    }
  }
}
```

These queries are typically sent over HTTP to a single server endpoint (unlike a REST architecture, where there are different endpoints for different resources).

The response is typically sent using JSON. For example, here's a sample response to the first query:
```
{
  "data": {
    "filterSongsByGenre": [
      {"title": "Song #0" },
      {"title": "Song #1" }
    ]
  }
}
```

In a similar way, you can define mutations to perform modifications on these types (optionally defining `Input` types):
```
# ...

input SongInput {
   title: String!
   genre: String!
   duration: Int!
   artistID: ID!
}

type SongMutations {
   newSong(song: SongInput!): Song
}

schema {
   query: SongQueries
   mutation: SongMutations
}
```

Here's a sample mutation based on the above declaration:
```
mutation {
  newSong(song: {
    title: "Song #2",
    duration: 122,
    genre: "hrock",
    artistID: 1,
  }) {
    id
    title
    genre
    artist {
      name
    }
  }
}
```

Beyond the convention of using mutations to modify data, an important distinction between queries and mutations is that queries are executed in parallel, while mutations are executed one after the other to avoid race conditions.

There's a lot more to [learn about GraphQL](http://graphql.org/learn/), but these concepts (types, queries, and mutations) are the foundation.

Now let's see how Spring Boot can help us implement a GraphQL server in an easy way.

# Spring Boot support for GraphQL

[graphql-java](http://graphql-java.com) is a Java library that implements the GraphQL specification.

You can add the library as a dependency on your project to start using it. At the time of writing this tutorial, the latest version is `6.0`:
```
<!-- https://mvnrepository.com/artifact/com.graphql-java/graphql-java -->
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java</artifactId>
    <version>6.0</version>
</dependency>
```

However, you'll also want to use a library like [GraphQL Java Tools](https://github.com/graphql-java/graphql-java-tools) to parse GraphQL schemas instead of describing your types programmatically. This library also maps them automatically to Java objects:
```
<!-- https://mvnrepository.com/artifact/com.graphql-java/graphql-java-tools -->
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-tools</artifactId>
    <version>4.3.0</version>
</dependency>

```

If you're building a web application, you'll also want to use [GraphQL Servlet](https://github.com/graphql-java/graphql-java-servlet), which implements a servlet that supports `GET` and `POST` requests for GraphQL queries:
```
<!-- https://mvnrepository.com/artifact/com.graphql-java/graphql-java-servlet -->
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-servlet</artifactId>
    <version>4.7.0</version>
</dependency>
```

Integrating all these projects requires a lot of boilerplate code. Luckily, `graphql-java` is supported by Spring Boot with the use of the [graphql-spring-boot-starter](https://github.com/graphql-java/graphql-spring-boot) project.

You just have to add `graphql-spring-boot-starter` to your project:
```
<!-- https://mvnrepository.com/artifact/com.graphql-java/graphql-spring-boot-starter -->
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-spring-boot-starter</artifactId>
    <version>3.10.0</version>
</dependency>
```

This starter will add and autoconfigure a GraphQL Servlet at `/graphql` and use a GraphQL schema library (like GraphQL Java Tools) to parse all the schema files found on the classpath.

Also, the following parameters will be available (via `application.yml` or `application.properties`) with these default values:
```
graphql:
  servlet:
    mapping: /graphql
    enabled: true
    corsEnabled: true
```

However, at the time of this writing, `graphql-spring-boot-starter` only works with Spring Boot 1.x, there's no support for Spring Boot 2 at the moment. According to this [issue](https://github.com/graphql-java/graphql-spring-boot/issues/40), support for Spring Boot 2 will be added when its general availability version is released ([due by February 2018](https://github.com/spring-projects/spring-boot/milestones) at this moment).

Now let's bootstrap a Spring Boot application using Spring Initializr.

# Setting up the project

Go to [https://start.spring.io/](https://start.spring.io/) and choose the following options:
- Maven (or Gradle)
- Java
- Spring Boot 1.5.9
- `com.example` (or any other identifier) as the Group
- `DemoGraphQL` (or any other identifier) as the Artifact
- As dependencies:
  - Web
  - H2
  - JPA
 
![Spring Initializr](https://raw.githubusercontent.com/pluralsight/guides/master/images/b57c485d-0b78-456c-99b5-61723898601a.jpg)

Generate the project, unzip the downloaded file, and open the project in your favorite IDE.

I'm going to use Java 9 so in `pom.xml`, let's start by changing the java version in the properties:
```
<properties>
  ...
  <java.version>9</java.version>
</properties>
```

At this time, Spring Boot 1.x has some [compatibility issues with Java 9](https://blog.frankel.ch/migrating-to-java-9/1/#gsc.tab=0). So we're going to remove `spring-boot-devtools` (if it's added to your project) and add the dependency `jaxb-api`:
```
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
  <version>2.3.0</version>
</dependency>
```

Next, add the `graphql-spring-boot-starter`:
```
<dependency>
  <groupId>com.graphql-java</groupId>
  <artifactId>graphql-spring-boot-starter</artifactId>
  <version>3.10.0</version>
</dependency>
```

It will add `graphql-java` and `graphql-java-servlet` as dependencies of your project, however, you still have to add a library to parse the GraphQL Schema. At this time, you have two options:
- [GraphQL Java Tools](https://github.com/graphql-java/graphql-java-tools)
- [GraphQL Spring Common](https://github.com/oembedler/spring-graphql-common)

I'll add GraphQL Java Tools:
```
<dependency>
  <groupId>com.graphql-java</groupId>
  <artifactId>graphql-java-tools</artifactId>
  <version>4.3.0</version>
</dependency>
```

For this tutorial, we're going to use an embedded H2 database, but since we removed `spring-boot-devtools`, we have to enable an embedded H2 database explicitly in `src/main/resources/application.resources`:
```
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

Update your Maven configuration in your IDE to download the dependencies if you haven't already, and run the main class of your project, in my case, `com.example.DemoGraphQL`. Alternatively, in a terminal window, you can execute `mvnw spring-boot:run` (or `gradle bootRun` if you're using Gradle).

Apart from some warnings, everything should look fine in the console logs. You can go to [http://localhost:8080/h2-console/login.jsp](http://localhost:8080/h2-console/login.jsp) and enter the following information:
- JDBC URL: `jdbc:h2:mem:testdb`
- User Name: `sa`
- Password: (empty)

![H2 console login](https://raw.githubusercontent.com/pluralsight/guides/master/images/166fd721-9d3d-4e15-b2bc-e527d365a862.png)

When you press the *Connect* button, the console of the in-memory database should appear:

![H2 console](https://raw.githubusercontent.com/pluralsight/guides/master/images/365b3266-f00a-40e5-ac21-44b498e95022.png)

If everything is fine, you're ready to start writing the GraphQL schema.

# Writing the Schema

At this point, if you go to [http://localhost:8080/graphql](http://localhost:8080/graphql), you'll see an error because nothing has been defined. So let's start with the GraphQL schema.

As said before, we're going to use the [DSL language](http://graphql-java.readthedocs.io/en/v6/schema.html) to create the schema instead of the programmatic way.

These schemas are written in `.graphqls` files, which can be placed anywhere in the classpath. So in `src/main/resources/` create a `graphql` directory and inside of it, create the files `author.graphqls` and `book.graphqls`:

![resources directory](https://raw.githubusercontent.com/pluralsight/guides/master/images/aeb1e1b3-0018-418e-a4d1-b7af8133d6c3.png)

You can split up your schema into more than one file to keep it organized. However, there can be only one root `Query` and only one root `Mutation` types that will contain all the query and mutation operations.

So in the file `author.graphqls`, we're going to define the `Author` type and the query and mutation operations we're going to implement for this type:
```
type Author {
  id: ID!
  firstName: String!
  lastName: String!
}

type Query {
  findAllAuthors: [Author]!
  countAuthors: Long!
}

type Mutation {
  newAuthor(firstName: String!, lastName: String!) : Author!
}
```

And in the file `book.graphqls`, in addition to the `Book` type, we're going to *extend* the `Query` and `Mutation` types:
```
type Book {
    id: ID!
    title: String!
    isbn: String!
    pageCount: Int
    author: Author
}

extend type Query {
    findAllBooks: [Book]!
    countBooks: Long!
}

extend type Mutation {
    newBook(title: String!, isbn: String!, pageCount: Int, author: ID!) : Book!
    deleteBook(id: ID!) : Boolean
    updateBookPageCount(pageCount: Int!, id: ID!) : Book!
}
```

This way, at runtime, the `Query` type will contain all the author and book operation, as it was defined like:
```
type Query {
    findAllAuthors: [Author]!
    countAuthors: Long!
    findAllBooks: [Book]!
    countBooks: Long!
}
```

Just like the `Mutation` type:
```
type Mutation {
    newAuthor(firstName: String!, lastName: String!) : Author!
    newBook(title: String!, isbn: String!, pageCount: Int, author: ID!) : Book!
    deleteBook(id: ID!) : Boolean
    updateBookPageCount(pageCount: Int!, id: ID!) : Book!
}
```

About the types you can assign to fields or to the return values of operations, `graphql-java` supports the following types:
- Scalar:
  - String
  - Boolean
  - Int
  - Float
  - ID
  - Long
  - Short
  - Byte
  - Float
  - BigDecimal
  - BigInteger
- Object (like the `Book` or `Author` types defined above)
- Interface, for example:
  ```
  interface Shape {
    color: String!,
    size: Int!
  }
  type Triangle implements Shape { }
  type Square implements Shape { }
  type Query {
    searchByColor(color: String!): [Shape]!
  }
  ```
- Union
  ```
  type Triangle {
    # fields ...
  }
  type Square {
    # fields ...
  }
  union Shape = Triangle | Square
  type Query {
    searchByColor(sizes: Int!): [Shape]!
  }
  ```
- InputObject
  ```
  input BookInput {
    title: String!
    genre: String!
    duration: Int!
    artistID: ID!
  }
  ```
- Enum
  ```
  enum Shape {
    TRIANGLE
    SQUARE
  }
  ```
Of course, you can also define your own types. See how the [Scalar types](https://github.com/graphql-java/graphql-java/blob/master/src/main/java/graphql/Scalars.java) are defined to give you an idea.

Once you have a type defined, the library GraphQL Java Tools will map it to a Java object. 

So let's create the package `model` and define an `Author` class with the constructors, getters and setters, `equals`, and `hashCode` methods:
```java
import javax.persistence.*;

@Entity
public class Author {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private String firstName;

    private String lastName;

    public Author() {
    }

    public Author(Long id) {
        this.id = id;
    }

    public Author(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Author author = (Author) o;

        return id.equals(author.id);
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }

    @Override
    public String toString() {
        return "Author{" +
                "id=" + id +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }
}
```

The same applies to the `Book` class:
```java
import javax.persistence.*;

@Entity
public class Book {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private String title;

    private String isbn;

    private int pageCount;

    @ManyToOne
    @JoinColumn(name = "author_id",
            nullable = false, updatable = false)
    private Author author;

    public Book() {
    }

    public Book(String title, String isbn, int pageCount, Author author) {
        this.title = title;
        this.isbn = isbn;
        this.pageCount = pageCount;
        this.author = author;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }

    public int getPageCount() {
        return pageCount;
    }

    public void setPageCount(int pageCount) {
        this.pageCount = pageCount;
    }

    public Author getAuthor() {
        return author;
    }

    public void setAuthor(Author author) {
        this.author = author;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Book book = (Book) o;

        return id.equals(book.id);
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }

    @Override
    public String toString() {
        return "Book{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", isbn='" + isbn + '\'' +
                ", pageCount=" + pageCount +
                ", author=" + author +
                '}';
    }
}
```

Notice that these classes are annotated `@Entity`, so they will represent the database's tables.

For scalar fields, getter and setter methods will be enough. However, for fields with complex types (like `author` in `Book`), you have to use `Resolver` objects to *resolve* the value of those fields.

`Resolver` objects implement the interface `GraphQLResolver`. Create the package `resolver`, and inside of it, the resolver for the `Book` class:
```java
public class BookResolver implements GraphQLResolver<Book> {
    private AuthorRepository authorRepository;

    public BookResolver(AuthorRepository authorRepository) {
        this.authorRepository = authorRepository;
    }

    public Author getAuthor(Book book) {
        return authorRepository.findOne(book.getAuthor().getId());
    }
}
```

GraphQL Java Tools works with four types of `Resolver` classes:
- `GraphQLResolver<T>` to resolve complex types.
- `GraphQLQueryResolver` to define the operations of the root `Query` type.
- `GraphQLMutationResolver` to define the operations of the root `Mutation` type.
- `GraphQLSubscriptionResolver` to define the operations of the root `Subscription` type.

[Subscriptions](http://graphql-java.readthedocs.io/en/v6/subscriptions.html) allows you to subscribe to a reactive source. They won't be reviewed in this tutorial, but the repositories for the `Author` and `Book` entities, and the `GraphQLQueryResolver` and `GraphQLMutationResolver` classes will be in the next section.

# Implementing the API

Let's start by creating the repositories. Spring Boot makes it really easy to create CRUD repositories.

In the `repository` package, create the interface `AuthorRepository`:
```java
public interface AuthorRepository extends CrudRepository<Author, Long> { }
```
And the interface `BookRepository`:
```java
public interface BookRepository extends CrudRepository<Book, Long> { }
```

Spring Boot will generate an implementation with methods to find, save, count and delete for the `Author` and `Book` entities.

This way, implementing a `GraphQLQueryResolver` is straightforward. In the `resolver` package, create the class `Query`:
```java
public class Query implements GraphQLQueryResolver {
    private BookRepository bookRepository;
    private AuthorRepository authorRepository;

    public Query(AuthorRepository authorRepository, BookRepository bookRepository) {
        this.authorRepository = authorRepository;
        this.bookRepository = bookRepository;
    }

    public Iterable<Book> findAllBooks() {
        return bookRepository.findAll();
    }

    public Iterable<Author> findAllAuthors() {
        return authorRepository.findAll();
    }

    public long countBooks() {
        return bookRepository.count();
    }
    public long countAuthors() {
        return authorRepository.count();
    }
}
```

Creating the `Mutation` class requires a little more work but it's still straightforward. After injecting the author and book repositories in the constructor:
```java
public class Mutation implements GraphQLMutationResolver {
    private BookRepository bookRepository;
    private AuthorRepository authorRepository;

    public Mutation(AuthorRepository authorRepository, BookRepository bookRepository) {
        this.authorRepository = authorRepository;
        this.bookRepository = bookRepository;
    }
}
```

Based on the GraphQL schema, you need to define the methods to create new authors and books, taking the declared parameters and saving the entities using the corresponding repositories:
```java
public class Mutation implements GraphQLMutationResolver {
    // ...
    public Author newAuthor(String firstName, String lastName) {
        Author author = new Author();
        author.setFirstName(firstName);
        author.setLastName(lastName);

        authorRepository.save(author);

        return author;
    }

    public Book newBook(String title, String isbn, Integer pageCount, Long authorId) {
        Book book = new Book();
        book.setAuthor(new Author(authorId));
        book.setTitle(title);
        book.setIsbn(isbn);
        book.setPageCount(pageCount != null ? pageCount : 0);

        bookRepository.save(book);

        return book;
    }
}
```

Notice that the GraphQL type `ID` can be converted to the Java types `String`, `Integer`, and `Long` and the parameters names of the method and the schema don't have to match. However, you have to take into account the number of parameters, their type, and whether they are optional or not.

Next, here's the delete method:
```java
public class Mutation implements GraphQLMutationResolver {
    // ...

    public boolean deleteBook(Long id) {
        bookRepository.delete(id);
        return true;
    }
}
```

And finally, the method to update the book's page count:
```java
public class Mutation implements GraphQLMutationResolver {
    // ...

    public Book updateBookPageCount(Integer pageCount, Long id) {
        Book book = bookRepository.findOne(id);
        book.setPageCount(pageCount);

        bookRepository.save(book);

        return book;
    }
}
```

However, there's something about this method is not quite right.

What if the ID passed as argument is an invalid one?

The method will throw a `NullPointerException` when the book to be updated cannot be found. This will be translated to a *Internal server error* on the client-side.

But the thing is that by default, any unhandled exception on the server-side will reach the client as a generic *Internal server error*.

Take a look at the [default GraphQL Servlet's error handler](https://github.com/graphql-java/graphql-java-servlet/blob/master/src/main/java/graphql/servlet/DefaultGraphQLErrorHandler.java):
```java
public class DefaultGraphQLErrorHandler implements GraphQLErrorHandler {
    // ...
  
    @Override
    public List<GraphQLError> processErrors(List<GraphQLError> errors) {
        final List<GraphQLError> clientErrors = filterGraphQLErrors(errors);
        if (clientErrors.size() < errors.size()) {

            // Some errors were filtered out to hide implementation - put a generic error in place.
            clientErrors.add(new GenericGraphQLError("Internal Server Error(s) while executing query"));

            errors.stream()
                .filter(error -> !isClientError(error))
                .forEach(error -> {
                    if(error instanceof Throwable) {
                        log.error("Error executing query!", (Throwable) error);
                    } else {
                        log.error("Error executing query ({}): {}", error.getClass().getSimpleName(), error.getMessage());
                    }
                });
        }

        return clientErrors;
    }
  
    // ...
}
```

Only the client errors (for example, when you misspell the name of a field) are processed. The rest of the errors are treated as a generic error and then logged.

If you want the client to get the correct message, the first thing you need to do is create a custom exception that implements `GraphQLError`. For example, a `BookNotFoundException` class:
```java
public class BookNotFoundException extends RuntimeException implements GraphQLError {

    private Map<String, Object> extensions = new HashMap<>();

    public BookNotFoundException(String message, Long invalidBookId) {
        super(message);
        extensions.put("invalidBookId", invalidBookId);
    }

    @Override
    public List<SourceLocation> getLocations() {
        return null;
    }

    @Override
    public Map<String, Object> getExtensions() {
        return extensions;
    }

    @Override
    public ErrorType getErrorType() {
        return ErrorType.DataFetchingException;
    }
}
```

`GraphQLError` provides a field called *extensions* to pass additional data to the error object send to the client. In this case, we'll use it to pass the invalid book ID.

This way, in the `updateBookPageCount`method, if the book cannot be found, we throw this exception:
```java
public class Mutation implements GraphQLMutationResolver {
    // ...

    public Book updateBookPageCount(Integer pageCount, Long id) {
        Book book = bookRepository.findOne(id);
        if(book == null) {
            throw new BookNotFoundException("The book to be updated was not found", id);
        }
        book.setPageCount(pageCount);

        bookRepository.save(book);

        return book;
    }
}
```

By using an exception as a GraphQLError, the client will receive the complete stack trace in addition to the exception message.

If you don't want this behavior, you can create an adapter class to *hide* the exception (which can be wrapped in an `ExceptionWhileDataFetching` class):
```java
public class GraphQLErrorAdapter implements GraphQLError {

    private GraphQLError error;

    public GraphQLErrorAdapter(GraphQLError error) {
        this.error = error;
    }

    @Override
    public Map<String, Object> getExtensions() {
        return error.getExtensions();
    }

    @Override
    public List<SourceLocation> getLocations() {
        return error.getLocations();
    }

    @Override
    public ErrorType getErrorType() {
        return error.getErrorType();
    }

    @Override
    public List<Object> getPath() {
        return error.getPath();
    }

    @Override
    public Map<String, Object> toSpecification() {
        return error.toSpecification();
    }

    @Override
    public String getMessage() {
        return (error instanceof ExceptionWhileDataFetching) ? ((ExceptionWhileDataFetching) error).getException().getMessage() : error.getMessage();
    }
}
```

So now the only thing missing is to redefine GraphQL's default error handler.

You can create a Spring `@Configuration` class or use the main class of the application annotated with `@SpringBootApplication` to create a bean of type `GraphQLErrorHandler` to replace the default error handler implementation:
```java
@SpringBootApplication
public class DemoGraphQlApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoGraphQlApplication.class, args);
	}
	
	@Bean
	public GraphQLErrorHandler errorHandler() {
		return new GraphQLErrorHandler() {
			@Override
			public List<GraphQLError> processErrors(List<GraphQLError> errors) {
				List<GraphQLError> clientErrors = errors.stream()
						.filter(this::isClientError)
						.collect(Collectors.toList());

				List<GraphQLError> serverErrors = errors.stream()
						.filter(e -> !isClientError(e))
						.map(GraphQLErrorAdapter::new)
						.collect(Collectors.toList());

				List<GraphQLError> e = new ArrayList<>();
				e.addAll(clientErrors);
				e.addAll(serverErrors);
				return e;
			}

			protected boolean isClientError(GraphQLError error) {
				return !(error instanceof ExceptionWhileDataFetching || error instanceof Throwable);
			}
		};
	}
}
```

If you don't mind showing the exception stack trace to your clients, you can just return the list of errors that the method `processErrors` receives as an argument.

But in this case, we do. So we first collect the client errors as they are, then the server errors, converting them to the adapter type (GraphQLErrorAdapter), and finally returning a list of all the errors collected.

While we're at it, let's also declare as Spring beans the resolvers for the `Book`, `Query`, and `Mutation` types:
```java
@SpringBootApplication
public class DemoGraphQlApplication {
    // ...
    
    @Bean
	public BookResolver authorResolver(AuthorRepository authorRepository) {
		return new BookResolver(authorRepository);
	}

	@Bean
	public Query query(AuthorRepository authorRepository, BookRepository bookRepository) {
		return new Query(authorRepository, bookRepository);
	}

	@Bean
	public Mutation mutation(AuthorRepository authorRepository, BookRepository bookRepository) {
		return new Mutation(authorRepository, bookRepository);
	}
}
```

And if you want, with a `CommandLineRunner` bean, you can insert some data into the database:
```java
@SpringBootApplication
public class DemoGraphQlApplication {
    // ...
    
    @Bean
	public CommandLineRunner demo(AuthorRepository authorRepository, BookRepository bookRepository) {
		return (args) -> {
			Author author = new Author("Herbert", "Schildt");
			authorRepository.save(author);

			bookRepository.save(new Book("Java: A Beginner's Guide, Sixth Edition", "0071809252", 728, author));
		};
	}
}
```

And that's it. Let's test the application.


# Testing with GraphiQL

Run the application and go to [http://localhost:8080/graphql/schema.json](http://localhost:8080/graphql/schema.json). You should see a description of the GraphQL schema, something like this:
```
{
    "data": {
        "__schema": {
            "queryType": {
                "name": "Query"
            },
            "mutationType": {
                "name": "Mutation"
            },
            "subscriptionType": null,
            "types": [
                {
                    "kind": "OBJECT",
                    "name": "Query",
                    "description": "",
                    "fields": [
                        {
                            "name": "findAllAuthors",
                            "description": "",
                            "args": [],
                            "type": {
                                "kind": "NON_NULL",
                                "name": null,
                                "ofType": {
                                    "kind": "LIST",
                                    "name": null,
                                    "ofType": {
                                        "kind": "OBJECT",
                                        "name": "Author",
                                        "ofType": null
                                    }
                                }
                            },
                            "isDeprecated": false,
                            "deprecationReason": null
                        },
                        ....
                    ]
                }
            ]
        }
    }
}
```

Now you have three options to test the GraphQL server.

As the `/graphql` endpoint is exposed via HTTP, you can use [curl](https://curl.haxx.se/) or [Postman](https://www.getpostman.com/).

For example, if you want to execute this query:
```
{
 findAllBooks {
   id
   title
 }
}
```

The request could be sent via HTTP POST as JSON in this way:
```
{
	"query":"{findAllBooks { id title } }"
}
```

Here's how it looks in Postman:

![Postman POST](https://raw.githubusercontent.com/pluralsight/guides/master/images/454d3e18-7fb4-4323-962e-7c44124e5404.jpg)

You can also send a GET request URL encoded, so a query like:
```
http://localhost:8080/graphql?query={findAllBooks{id title}}
```

It will have to be sent like:
```
http://localhost:8080/graphql?query=%7BfindAllBooks%7Bid%20title%7D%7D
```

![Postman GET](https://raw.githubusercontent.com/pluralsight/guides/master/images/b1255502-0617-4dd3-98c1-5c87d7f03423.jpg)

But as you can see, this could become a painful process when working with complex queries.

Luckily, we can use [GraphiQL](https://github.com/skevy/graphiql-app), an Electron app that you can install to test your GraphQL endpoint:

![GraphiQL electron](https://raw.githubusercontent.com/pluralsight/guides/master/images/1d6e1ba2-2333-471e-9539-9e9cc6c46a80.jpg)

There's a documentation tab on the right where you can see all the queries and mutations available and the app even has autocompletion for types and fields:

![GraphiQL doc and autocompletion](https://raw.githubusercontent.com/pluralsight/guides/master/images/e7a8bfc6-51ce-4ae2-9537-5de92035d7ed.jpg)

If you don't want to install the app, there's a web version you can incorporate into your application.

There's a Spring Boot starter for this version. Just add the following dependency:
```
<!-- https://mvnrepository.com/artifact/com.graphql-java/graphiql-spring-boot-starter -->
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphiql-spring-boot-starter</artifactId>
    <version>3.10.0</version>
</dependency>
```

Save the changes and run the application again. Now go to [http://localhost:8080/graphiql](http://localhost:8080/graphiql) and execute a query, for example:
```
{
  findAllBooks {
    id
    isbn
    title
    pageCount
    author {
      firstName
      lastName
    }
  }
}
```

The result should be something like:

![GraphiQL web](https://raw.githubusercontent.com/pluralsight/guides/master/images/3dd7a43f-fb89-466b-b5fa-550f5df8780e.jpg)

These are the available Spring Boot configuration parameters for GraphiQL with their default values:
```
graphiql:
    mapping: /graphiql
    endpoint: /graphql
    enabled: true
```

This means that if you change the endpoint where the GraphQL servlet is deployed, you'll also have to change the GraphiQL configuration (in `application.properties`:
```
graphql.servlet.mapping: /graphql2

graphiql.mapping: /graphiql
graphiql.endpoint: /graphql2
```

Otherwise, you'll get an error like the following:
```
{
  "timestamp": 1513702998134,
  "status": 404,
  "error": "Not Found",
  "message": "No message available",
  "path": "/graphql2"
}
```

Now try some queries.

For example, to create a new book:
```
mutation {
  newBook(
    title: "Java: The Complete Reference, Tenth Edition", 
    isbn: "1259589331", 
    author: 1) {
      id title
  }
}
```

To update the page count of a book:
```
mutation {
  updateBookPageCount(pageCount: 1344, id: 2) {
    id pageCount
  }
}
```
To delete a book:
```
mutation {
  deleteBook(id:2)
}
```

You can also watch the modifications performed by these mutations on the database. And don't forget to try the case where there's no book to update:
```
mutation {
  updateBookPageCount(pageCount: 1344, id: 20) {
    id pageCount
  }
}
```

# Conclusion

GraphQL is a flexible and powerful alternative to REST for building APIs. Here are some points to remember:
- GraphQL is a query language that is independent of the source of the data (database, web service, etc.).
- GraphQL describes the data that an API provides via a strong typing system.
- Through a schema, GraphQL declares the resources, relationships between them, and operations available.
- All operations go through a single endpoing that can be accessed through HTTP.

There's a lot more to learn about GraphQL, of course. Here are some good resources:
- [graphql-java documentation](http://graphql-java.readthedocs.io/)
- [GrapQL official documentation](http://graphql.org/learn/)
- [How to GraphQL](https://www.howtographql.com/basics/0-introduction/)

You can find the entire source code of the project [on GitHub](https://github.com/eh3rrera/graphql-java-spring-boot-example).

Contact me if you have questions or problems. Thanks for reading.
