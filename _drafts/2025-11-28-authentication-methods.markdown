---
layout: post
title: "Authentication Methods"
date: 2025-11-28 00:00:00 -0700
categories: tech
tags: python
---

## Basic Authentication

Basic Auth is one of the oldest authentication methods for the web. The client
takes the following acions:

  1. Collect the user's username and password
  2. Concatenate the username and password with a colon
  3. Encode the string with Base64
  4. Send in the Authorization header with Bearer, as `Authorization: Bearer XXXXXXXX`, where `XXXXXXXX` is the Base64 encoded string `username:password`

## Session Authentication

  1. Client sends credentials to the server
  2. Server validates the credentials and creates a session
  3. Server sends a unique Session ID to the client in an HttpOnly cookie
  4. On subsequent requests the client sends back the cookie
  5. Server uses the Session ID to authenticate the user and look up data related to the session
  
## SSO Authentication


