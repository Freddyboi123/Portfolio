---
date: 2026-04-10T15:24:04+02:00
draft: false
title: 'Hashing'
---

# What is Hashing
When it comes to user security, hashing is an extremely powerful tool that we, as developers, use.

Hashing is the process of taking a human-readable string and transforming it into a sequence of seemingly random characters, known as a hash. Unlike encryption, hashing is a one-way operation, meaning the original value cannot be reconstructed from the hash.

Instead, computers use hashing to verify data by comparing hashes. If two inputs produce the same hash, they are considered a match.

Let’s invent an imaginary accounts.

Without hashing, those accounts would look like this in our database:

![ImageFromBD](../image-1.png)

As you can see, all the passwords are sitting in plain text—completely exposed.

This introduces serious vulnerabilities into the system.

In the event of a data breach, every user’s credentials would be instantly compromised, forcing a platform-wide password reset.

It also opens the door for insider threats, where a malicious developer could easily access sensitive user information.

And perhaps worst of all, it increases the likelihood of accidental leaks—where passwords are unintentionally exposed during routine operations like logging or data retrieval.


If we instead hash all our passwords, this is what the database would look like:
![alt text](../image-2.png)

At this point, the data is no longer human-readable—it’s just a string of seemingly random characters.

Even if an attacker gains access to the database, they won’t see actual passwords—only hashes. And with a properly designed hashing system, reversing those hashes back into the original passwords is computationally infeasible.

That’s why hashing isn’t optional. it’s a fundamental requirement for securely storing passwords.

# Salt 
While hashing is a powerful way to protect passwords, it’s not enough on its own.

If two users have the same password, they will also have the same hash. This creates a vulnerability, as attackers can use precomputed tables (such as rainbow tables) to reverse commonly used passwords.

This is where salting comes in.

A salt is a unique, randomly generated value that is added to each password before it is hashed. This ensures that even if two users have the same password, their hashes will be completely different.

For example, without salting:

password123 → hashA
password123 → hashA

With salting:

password123 + salt1 → hashX.

password123 + salt2 → hashY.

As you can see, the same password now produces different hashes because each user has a unique salt.

This makes it significantly harder for attackers to crack passwords, as they can no longer rely on precomputed hash tables or easily detect users with identical passwords.

When a user logs in, the system retrieves the stored salt, combines it with the provided password, hashes the result, and compares it to the stored hash.

It is important to understand that the salt is not decrypted. Instead, it is stored and reused during login to recreate the same hashing conditions.

By combining hashing with salting, we add another critical layer of security, making our system far more resilient against attacks.


## My implimentation
The way I chose to implement hashing and salting in my project was to build it directly into the User `class` constructor.

This means that the moment I create a new `User`, the password is automatically hashed before it is ever stored.
```java
User user1 = new User("Frederik","DerErEtYndightLand","fred@dk.dk");
```

The constructor `new User(String name, String password, String email)` immediately converts the password into a hash:

```java
    public User(String username, String password, String email){
        String salt = BCrypt.gensalt(12);
        String hashedPassword = BCrypt.hashpw(password,salt);
        this.username = username;
        this.email = email;
        this.password =hashedPassword;
        addRole(Roles.USER);
    }
```

As you can see, it doesn’t take much to implement this essential security feature.

The raw password is never stored. Instead, we generate a salt and pass both the password and the salt into the hashing function, which produces a secure hash.

With this approach, security is enforced by design—there is no way to accidentally store a plain-text password, because hashing happens the moment a user is created.

# verification
During login, I verify the user’s password by retrieving the stored user from the database and using BCrypt.checkpw() to compare the entered password with the stored hash.

The verification logic is encapsulated inside the User class:
```java
 public boolean verifyPassword(String password){
        return BCrypt.checkpw(password,this.password);
    }
```
This keeps the authentication logic clean and ensures that password handling remains consistent across the application.

In the service layer, I fetch the user by email and then call this method:

```java
public User getVerifiedUser(String email, String password){
    try(EntityManager em = emf.createEntityManager()){
        em.getTransaction().begin();
        TypedQuery<User> query = em.createQuery(
            "SELECT u FROM User u WHERE u.email = :userEmail", User.class
        );
        query.setParameter("userEmail", email);
        User foundUser = query.getSingleResult();

        if (foundUser.verifyPassword(password)){
            return foundUser;
        } else {
            throw new ValidationException(Map.of());
        }
    } catch (NoResultException e) {
        System.out.println("No user was found with the email: " + email);
        return null;
    }
}
```
BCrypt.checkpw() automatically extracts the salt from the stored hash, re-hashes the entered password using that salt, and compares the result.

This means I never have to manually handle salts or compare raw passwords—everything is handled securely by the library.
