---
date: 2026-04-10T21:23:56+02:00
draft: false
title: 'REST'
---

# What is a REST Api
A REST API (Representational State Transfer Application Programming Interface) is a way for different software systems to communicate with each other over the internet. It follows a set of architectural principles that make communication simple, predictable, and scalable.

At its core, a REST API exposes resources—such as users, posts, or comments—through URLs. These resources can then be interacted with using standard HTTP methods:

GET to retrieve data
POST to create data
PUT/PATCH to update data
DELETE to remove data

A key characteristic of REST APIs is that they are stateless, meaning each request contains all the information needed for the server to process it. The server does not store any memory of previous requests.

Most REST APIs exchange data in JSON format, making them easy to work with across different programming languages and platforms.

# Why do we use REST Api
REST APIs are widely used because they provide a clean and efficient way to connect systems. In modern applications, the frontend (what users see) and backend (where data and logic live) are often separated—and REST APIs act as the bridge between them.

Some key reasons we use REST APIs include:

Scalability
- Because requests are stateless, servers can handle many clients without needing to track sessions.

Flexibility
- Different clients (web apps, mobile apps, third-party services) can all use the same API.

Simplicity
- REST relies on standard HTTP methods, making it easy to understand and implement.

Separation of concerns
- Frontend and backend can be developed independently, improving development speed and maintainability.

Interoperability
- Systems written in different languages or running on different platforms can still communicate seamlessly.


In short, REST APIs make it possible to build modular, reusable, and scalable systems

# How i have i implimented REST Api
In my project, I have implemented a REST-style API to handle communication between the client and the server. The system is structured around different resources, each managed by its own controller.

For example:

A UserController handles user-related operations
A PostController manages posts
A CommentController deals with comments
A FriendshipController manages relationships between users

These controllers are connected through a central routing class, where endpoints are defined and mapped to specific actions.

Each route corresponds to a specific operation, such as retrieving data, creating new entries, or updating existing ones. Access control is also enforced using roles, ensuring that only authorized users can perform certain actions.