---
date: '2026-04-01T13:35:52+02:00'
draft: false
title: 'Builder'
---

# working with Lombok

Sometimes working with `Lombok` is a bittersweet experience. In many ways, it offers solutions that make the day-to-day life of a programmer easier.

However, it can also introduce unexpected problems due to the dependency.

This happened to me today while creating a new populator class for my application to seed the database. As I was refactoring my setup for creating users, I decided to implement a Lombok builder (because it’s different, so it must be better).

This decision led to an interesting problem.'

After setting up the basic `createUsers` part, I ran a test and immediately noticed something was wrong.

![Builder Error Image](/images/builderErrorFirst.png)

As you can see, my passwords are no longer hashed. 
After my inisial confusion, i desided to look at my lombok setup 
- *if in doubt, blame lombok*

I realized that the only thing i had changed that could effect this, was that i went from using a default constructor

```java
   User user3 = new User ("Luke","AlingeSandvig","luke@dk.dk");

```
To a lombok Builder

```java
 User user3 = User.builder().username("Luke").password("AlingeSandvig").email("luke@dk.dk").build();
```

After doing some research, I came to understand that the `Lombok builder` does not use the constructor I created. Instead, it bypasses it and constructs the object directly (effectively behaving like a no-args construction plus field assignment), which explains why my constructor was ingnored and the password hashing logic was never executed.

The fix was simple, don't use the Builder, and rebase the code on a simple constructor.
```java
        User user1 = new User("Frederik","SolOverGudhejm","fred@dk.dk");
        User user2 = new User ("Daniel","RønneStrand","dan@dk.dk");
        User user3 = new User ("Luke","AlingeSandvig","luke@dk.dk");
        User user4 = new User("Emil","VivaLaFrancs","Emil@dk.dk");
```

And as you can see this worked as expected 

![Builder Error Image](/images/errorSolved.png)


# What i learned
1: Sometimes it’s best to keep things simple and readable.
Yes, `Lombok` is a great tool, but it isn’t suited for every situation.

Writing clean and reliable code often comes from simplicity rather than complexity.

2: Keep testing! just because it worked before does not mean that it will work in 5 min, test test test!
