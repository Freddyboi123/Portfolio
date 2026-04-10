---
date: 2026-04-10T17:33:26+02:00
draft: false
title: 'JPA_ORM'
---


# ORM
ORM stands for Object-Relational Mapping.

It is a technique that allows us to map objects in our code directly to tables in a relational database.

Instead of writing raw SQL queries, we can work with Java objects, and the ORM framework handles the database interaction for us.

This means we can think in terms of objects and relationships, rather than tables and joins, which makes the code easier to read, write, and maintain.

# JPA
JPA (Java Persistence API) is a specification that defines how ORM should be implemented in Java.

It provides a standard way to:

Store objects in a database
Retrieve objects from a database
Manage relationships between objects

In practice, we have implemented JPA by using Hibernate.

# JPQL
By using JPA and ORM, instead of writing SQL, we use JPQL (Java Persistence Query Language), which works with objects.

This means that instead of having to think about our data both as POJOs and database entities, we can primarily think in terms of our Java objects.

This greatly simplifies development, as it reduces many of the common mistakes associated with writing raw SQL—especially those that only surface at runtime.

With JPQL and Hibernate, queries are validated against our entity model. If we try to access fields or relationships that do not exist, the framework will throw an error, helping us catch issues earlier in development.

For example, attempting to query unrelated data—such as retrieving a phone number from a comment entity—will result in an error, because that relationship is not defined in the model.



```java
TypedQuery<User> query = em.createQuery(
    "SELECT u FROM User u WHERE u.email = :email", User.class
);
query.setParameter("email", email);
```

As you can see from this example, we are querying for a `User` with a specific email.

This works because the `User` entity (our POJO) contains the field `private String email`, which is mapped to a column in the database.

In JPQL, we don’t query tables and columns directly—we query entities and their fields. This means `u.email` refers to the `email` field in the `User` class, not a column name in the database.


# How we impliment Hibernate (ORM)

Hibernate primarily needs to know two things:

- How to access the database
- Which entities it should manage

Setting up Hibernate to connect to a database can be a bit complex (and not the most exciting thing to write about), but the TL;DR version is this:

When running the program locally, we use the following method to configure the database connection:

```java
private static void setDevProperties(Properties props) {
        String dbName = Utils.getPropertyValue("DB_NAME", "config.properties");
        String username = Utils.getPropertyValue("DB_USERNAME", "config.properties");
        String password = Utils.getPropertyValue("DB_PASSWORD", "config.properties");

        props.put("hibernate.connection.url", "jdbc:postgresql://localhost:5432/" + dbName);
        props.put("hibernate.connection.username", username);
        props.put("hibernate.connection.password", password);
    }
```
this looks at our projects config fill and finds the relevent infomartion to get acces to the database. 

if we are deployed we do almost the exact same thing 
```java
    private static void setDeployedProperties(Properties props) {
        String dbName = System.getenv("DB_NAME");
        props.setProperty("hibernate.connection.url", "jdbc:postgresql://db:5432/" + dbName);
        props.setProperty("hibernate.connection.username", System.getenv("DB_USERNAME"));
        props.setProperty("hibernate.connection.password", System.getenv("DB_PASSWORD"));
    }
```
Instead of reading from a local file, we retrieve the configuration from environment variables on the server (e.g., an Ubuntu server).

This is a more secure and flexible approach for production environments, as sensitive information like database credentials is not stored directly in the codebase.


Managing which entities Hibernate should handle—and how the relationships between them are defined—is another important part of the setup.

We register all the entities that we want Hibernate to manage in our `HibernateRegistry` class:
```java
we register all the entities that we are interested in hibernate maenging, in our HibernateRegestery class:
static void registerEntities(Configuration configuration) {
            configuration.addAnnotatedClass(User.class);
            configuration.addAnnotatedClass(PrivacySettings.class);
            configuration.addAnnotatedClass(Post.class);
            configuration.addAnnotatedClass(Comment.class);
            configuration.addAnnotatedClass(Friendship.class);
    }
```
Here, you can see that we have registered five classes that Hibernate will manage.

Each of these classes is annotated with @Entity, signaling to Hibernate that the class should be managed as a persistent entity and mapped directly to a database table.



In JPA, relationships between entities define how objects are connected and how data is structured in the database.

Using the User class, we can see several examples of how these relationships are implemented.


```java
@OneToOne(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
private PrivacySettings privacySettings;
```
This defines a one-to-one relationship between User and PrivacySettings.

This means:

- Each user has exactly one set of privacy settings
- Each PrivacySettings belongs to exactly one user

when we look at how the relationship is set up we see
- `mappedBy = "user"` → the PrivacySettings class owns the relationship
- `cascade = ALL` → operations on User (save, delete) affect PrivacySettings
- `orphanRemoval` = true → if the relationship is removed, the child is deleted


we also have the `User` relation to `Post`
```java
@OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
private Set<Post> posts = new HashSet<>();
```
This defines a one-to-many relationship:

- One User can have many Post objects
- Each Post belongs to one User

When creating a new post, we need to ensure that both sides of the relationship are properly set.

This is handled using a helper method:

```java
public void addPost(Post post) {
    if (posts == null) {
        posts = new HashSet<>();
    }
    posts.add(post);
    post.setUser(this);
}
```
This ensures that:

- The Post references the correct User
- The User contains the new Post in its collection

Keeping both sides in sync is essential for Hibernate to correctly persist the relationship.

# What we have acomplised. 
Implementing Hibernate in this way provides several important benefits.

First, it becomes much easier to create reliable and stable DAO classes that handle traffic to and from the database.

Secondly, we have built a network of relationships between our entities, allowing us to manage and manipulate the database with minimal risk.

For example, if I delete a user from the system, I don’t need to manually delete all of their posts, comments, and related data.

Because of the relationships and cascade settings we have defined, Hibernate handles this automatically.

When a User is deleted:

- All of their posts are deleted
- All of their comments are deleted
- Any comments made on their posts are also removed (through the relationships between entities)