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

This selects all users in the database and comments away the code for a password check, allowing us to bypass authentication. It seems like there is only 1 entry in the database as `LIMIT 1` is not needed in our exploit payload.

# Through The Gates
Past the login screen, we are presented with a COVID tracker showing the various number of cases at various locations across Singapore. There is also a search bar that allows us to filter these locations. Could it be, another SQL Injection?

![Covid Trakcer](/assets/images/covidtracker.png)

We immediately try the same payload in hopes that the POST data will be likewise used in an SQL query.

![Test SQLi](/assets/images/testsqli_1.png)

We note that all the waypoints are still present. Seems like it is indeed vulnerable to SQL injection. If it was blacklisted in any way, then it would not return all the waypoints correctly. This gives us a rough idea of what the query could be like.

```sql
SELECT locations, caseNums FROM covidTable WHERE locations = '<POST_DATA>'
```

## Leveraging ORDER BY 

With this in mind, my first thought process was to attempt to use `ORDER BY` to enumerate the number of columns. `ORDER BY` is a SQL clause that sorts a data set according to a particular column, which is specified via either the column name or an integer specifying the column order. By leveraging the latter, we can use errors to tell us which is the correct number of columns. This approach is known as error-based SQL injection!

![Order By Successful](/assets/images/orderby_1.png)

![Order By Failure](/assets/images/orderby_2.png)

The above 2 screenshots showcase how error-based SQL injection can be done. With the payload injected, the imagined SQL query becomes something similar to this:

```sql
SELECT locations,caseNums FROM covidTable WHERE locations ='' ORDER BY 3; --'
```

We see that we successfully got waypoints returned from 3, but none from 4, which suggests that the data structure has 3 columns: likely an ID, location and case number.

## UNION SELECT, or so I thought
Upon successful enumeration of the number of columns, my next thought process was to leverage the `UNION SELECT` clause to attempt to control one of the displayed fields to leak information from the database. 

`UNION SELECT` clauses have 2 necessary factors to run successfully:
- The number of columns returned by both `SELECT` statements must be the same 
- Data types of each columns must be the same 

Since we know the number of columns, I attempted to use the `UNION SELECT` clause as follows.

![Union Select](/assets/images/union_1.png)

However, we see that no waypoints poped up and instead we get an error message saying `loc.geo.split is not a function`. On hindsight, perhaps this was a sign of a type difference between the columns. However, instead of continuing to pursue this approach, I decided to pursue the error-based approach further, which will be detailed in the next section.


## Back to the Error-Based Approach
We first used the error-based approach to enumerate that there is a table called `flag` that has a column `value` containing the flag.

Once we've gained that information, we can then continue to use the error-based appoach to further enumerate the flag in value.

`' AND (SELECT hex(substr(value,10,1)) from flag limit 1 offset 0) = hex('w') --`

```sql
SELECT locations, caseNums FROM covidTable WHERE locations = '' AND (SELECT hex(substr(value,10,1)) from flag limit 1 offset 0) = hex('w') --
```

Let's dissect this payload further. The first thing to note is the usage of the `AND` clause. `AND` dictates that the clause that comes after the AND has to be TRUE in order for the entire query to be true. We can use this factor in our error-based approach.

The second part of the query contains the following `(SELECT hex(substr(value,10,1)) from flag limit 1 offset 0) = hex('w')`. This statement extracts out the first row from the value column character by character and allows us to verify that character. In this payload, we start with the character from the 10th position as we know that flags will start with `greyhats{`. Let's see this payload in action.

![Error Based](/assets/images/errorbased.png)

Here we can see that the waypoints are still visible, meaning that the second part of the payload is TRUE (ie. the 10th character of our flag is `w`). We can then use this approach to continue to 'brute force' the flag until we get it.

Final flag: `greyhats{w3bApp5_n33d_v@cc1ne?_4521f}`

# Remediations
SQL Injection vulnerablities arise when input that a user controls is inserted directly into a SQL query. 

Some remediations that can be put in place to prevent these vulnerabilities from arising is:

- The usage of a blacklist to filter for particular SQL keywords like `SELECT` or `AND`. One thing to note is that blacklists are often not 100% fullproof and creative threat actors can still find a way to bypass it.

- Usage of parametrized queries. Parametrized queries are pre-compiled SQL queries and area simple and effective way to prevent SQLi attacks. Parameterized queries run the query PRIOR to the user's input being inserted, hence the user's input is unable to effect other tables /data sets. 

# Conclusion
If you've made it to here, thank you for reading my humble writeup documenting some learnings about SQL injections. Look forward to more content on this site!

