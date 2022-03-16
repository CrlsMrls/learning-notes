# Web Security Model

## Same-origin policy

**Goal:** Two websites should not be able to tamper each other

**Strategy:** each resource is assigned origin.JS so it can only access resources from its own origin.

**Origin:** scheme + hostname + port

**Implementation:**

- each resource has client-side resources: DOM storage, DOM tree, JS resources, cookies, etc.
- each frame gets the origin from its URL
- Passive content gets zero authority

Different origin domains can communicate only through postMessage

`XMLHttpRequest` also follow the same origin policy, unless the cors header is enabled: `Access-Control-Allow-Origin: foo.com`

## Cross-site request forgery (CRSF)

This attack forces an end user to execute unwanted actions on a web application in which they’re currently authenticated.

## Cross-site Scripting (XSS)

XSS attacks are a type of injection, in which malicious scripts are injected into otherwise benign and trusted web sites.

Sanitize inputs to prevent it. In other words, by constraining expressivity you can improve security.

## Content Security Policy (CSP)

[CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross Site Scripting (XSS) and data injection attacks.

## SQL Injection

By injecting some SQL query into an untrasted variables, the attacker could execute malicious SQL queries in the DB.

To prevent it, you must sanitize input fields (escape dangerous chars)

Some data layers already implement sanitization

## Cookies stealing

Are sent in every http request
https://publicsuffix.org/list/
