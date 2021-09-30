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

![Login Panel](./assets/images/loginpanel.png)

This payload works because the user's POST input is being directly used in a SQL query that is likely of the following form.

``` mysql
SELECT user FROM users WHERE username = '<POST DATA>' AND password = '<POST DATA>';
```

You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

```ruby
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
```

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
