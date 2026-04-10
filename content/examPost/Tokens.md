---
date: 2026-04-10T19:03:30+02:00
draft: false
title: 'Tokens'
---

# Web Tokens
In this project, authentication is handled using JSON Web Tokens (JWT).

A token is a secure, self-contained piece of data that is issued to a user after successful authentication. It is then used to verify the user’s identity on subsequent requests.

A JSON Web Token is a compact string that contains encoded information about a user.

Instead of storing session data on the server, the token itself carries the necessary information to identify and authorize the user.


## Benefits:
Scalability
- No session storage is required on the server
Security
- Tokens are signed using a secret key, preventing tampering
Efficiency
- The client includes the token in each request, avoiding repeated login checks
Role-Based Access
- User roles stored in the token can be used to restrict access to certain endpoints

# how we implemented it 
When a user logs in, the system verifies their credentials:

```java
User userEntity = userDAO.getVerifiedUser(user.getEmail(), user.getPassword());
```
If authentication is successful, a token is generated

```java
String token = createToken(new UserDTO(
    userEntity.getUsername(),
    userEntity.getPassword(),
    userEntity.getRolesAsString(),
    userEntity.getEmail()
));
```
The token is created using environment-specific configuration:
```java
return tokenSecurity.createToken(user, ISSUER, TOKEN_EXPIRE_TIME, SECRET_KEY);
```
This includes:

- Issuer – identifies who created the token
- Expiration time – limits how long the token is valid
- Secret key – used to sign and verify the token

This token is returned to the client:

```java
ctx.status(200).json(node
    .put("token", token)
    .put("username", userEntity.getUsername()));
```

After a user logs in and receives a token, that token is used to secure all protected endpoints in the system.

This is handled using middleware in the request lifecycle, where each incoming request is validated before reaching the controller.

The server then:

- Verifies the token signature
- Extracts user information
- Grants or denies access based on roles

# When we get a request 
In the application configuration, all incoming requests pass through two security steps:
```java
config.routes.beforeMatched(securityController::authenticate);
config.routes.beforeMatched(securityController::authorize);
```
This ensures that:

- The user is authenticated (token is valid)
- The user is authorized (has the correct role)

```java
public void authenticate(Context ctx) {
    if (ctx.method().toString().equals("OPTIONS")) {
        ctx.status(200);
        return;
    }

    Set<String> allowedRoles = ctx.routeRoles()
        .stream()
        .map(role -> role.toString().toUpperCase())
        .collect(Collectors.toSet());

    if (isOpenEndpoint(allowedRoles))
        return;

    UserDTO verifiedTokenUser = validateAndGetUserFromToken(ctx);
    ctx.attribute("user", verifiedTokenUser);
}
```
During authentication:

- Preflight requests (OPTIONS) are ignored
- Open endpoints (no roles required) are skipped

For protected endpoints:
- The token is extracted from the request
- The token is validated (signature + expiration)
- User data is extracted from the token
- The user is attached to the request context

The token is validated by the method `validateAndGetUserFromToken` that has the method `validateToken`

```java
private UserDTO verifyToken(String token) {
    boolean IS_DEPLOYED = (System.getenv("DEPLOYED") != null);
    String SECRET = IS_DEPLOYED
        ? System.getenv("SECRET_KEY")
        : Utils.getPropertyValue("SECRET_KEY", "config.properties");

    try {
        if (tokenSecurity.tokenIsValid(token, SECRET) && tokenSecurity.tokenNotExpired(token)) {
            return tokenSecurity.getUserWithRolesFromToken(token);
        } else {
            throw new ApiException(403, "Token is not valid");
        }
    } catch (ParseException | TokenVerificationException e) {
        throw new ApiException(HttpStatus.UNAUTHORIZED.getCode(), "Unauthorized. Could not verify token");
    }
}
```
here the seceret key is checked to see if it is the same key as our system expets 
If the token is invalid or missing, access is denied.


next we go to the Authorication.
```java
public void authorize(Context ctx) {
    Set<String> allowedRoles = ctx.routeRoles()
        .stream()
        .map(role -> role.toString().toUpperCase())
        .collect(Collectors.toSet());

    if (isOpenEndpoint(allowedRoles))
        return;

    UserDTO user = ctx.attribute("user");
    if (user == null) {
        throw new ForbiddenResponse("No user was added from the token");
    }

    if (!userHasAllowedRole(user, allowedRoles))
        throw new ForbiddenResponse(
            "User was not authorized with roles: " + user.getRoles() +
            ". Needed roles are: " + allowedRoles
        );
}
```
During authorization:

- The system checks which roles are required for the endpoint
- The authenticated user is retrieved from the request context
- The user’s roles are compared against the required roles

If no matching role is found, access is denied.

If everything checks out and the user has the required role to access the endpoint, the request is allowed to proceed and the endpoint is executed.





