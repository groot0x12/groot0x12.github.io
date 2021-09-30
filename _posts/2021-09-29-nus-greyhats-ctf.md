---
title: "NUS Greyhats CTF 2021"
date: 2021-09-29T00:00:00-00:00
categories:
  - ctf-writeups
tags:
  - ctf
  - SQLi
  - web
---

This is a writeup of one of my favourite Web challenges from NUS Greyhats CTF 2021. This challenge was entitled "Covid Tracker" and was an Intermediate level challenge, and serves as a great recap for error-based SQL Injection.

# Landing 

The challenge first starts out with a link that brings us to a landing page containing a login portal. We immediately try a standard SQL injection payload for authentication bypass as seen from the payload below. 

![Login Panel](/assets/images/loginpanel.png)


This payload works because the user's POST input is being directly used in a SQL query that is likely of the following form.

``` sql
SELECT user FROM users WHERE username = '<POST DATA>' AND password = '<POST DATA>';
```

Sending the payload `' or 1=1; --` then results in the query being:
```sql
SELECT user FROM users WHERE username = '' or 1=1; --' AND password = '<POST DATA>';
```


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
