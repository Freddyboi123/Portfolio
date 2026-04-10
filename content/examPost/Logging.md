---
date: 2026-04-10T21:38:18+02:00
draft: false
title: 'Logging'
---

We have implemented a simple logging mechanism within our `login` method to track authentication attempts and improve observability of our system.
```java
    public void login(Context ctx) {
        UserDTO user = ctx.bodyAsClass(UserDTO.class);
        try{
            User userEntity = userDAO.getVerifiedUser(user.getEmail(), user.getPassword());
            String token = createToken(new UserDTO(userEntity.getUsername(),userEntity.getPassword(),userEntity.getRolesAsString(),userEntity.getEmail()));
            ObjectNode node = objectMapper.createObjectNode();

            ctx.status(200).json(node
                    .put("token",token)
                    .put("username",userEntity.getUsername()));
            logger.info("Successful login by user: {} Email: ({})", user.getUsername(), user.getEmail());
        } catch (ValidationException ex) {
            logger.info("Authentication attempt failed for user: {} Email: ({})", user.getUsername(), user.getEmail());
            throw new ApiException(401, ex.getMessage());
        }
    }
```

This method handles two primary scenarios:

- A successful login, where a token is generated and returned to the client
- A failed login, where an exception is thrown and the attempt is logged

In both cases, the system logs relevant information about the login attempt, including the username and email. This provides visibility into how the authentication system is being used.

By running the application in Docker, we can easily monitor these logs in real time. This makes it possible to:

- Track user activity
- Identify failed login attempts
- Detect suspicious patterns, such as repeated login failures that could indicate brute-force attacks

While this is a simple implementation, it forms a useful foundation for monitoring and security. More advanced features—such as rate limiting, account lockouts, or structured logging—could be added in the future to further strengthen the system.