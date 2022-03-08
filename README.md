# Hack the System

## Preparation
1. Install [Docker](https://www.docker.com/get-started)
2. Run `docker pull bkimminich/juice-shop`

## Running the container
1. Run `docker run --rm -p 3000:3000 bkimminich/juice-shop`
2. **On Mac:** Browse to http://localhost:3000 
3. **On Windows:** browse to http://192.168.99.100:3000 (if you are using docker-machine instead of the native docker installation)

## Explanation: Security by obscurity
We'll dive into security by obscurity.

## Explanation: SQL Injection
We'll start with a group session explaining SQL Injection.

## Demo
Time to see SQL Injection in action, demonstrate to obtain list of users.

## Lab 1: SQL Injection

### Assignment
Somewhere on the site there is a severe SQL Injection vulnerability, can you locate and exploit it?

<details>
  <summary>Hint 1</summary>


The vulnerability allows us to login to the system.


</details>

<details>
  <summary>Hint 2</summary>


If you are familiar with SQL, think of what a login query generally looks like. If you are not have a look at hint 3!


</details>

<details>
  <summary>Hint 3</summary>


The login query looks something like the query below and will log the user in if there is a match 

```SQL 
SELECT * from users where email = 'FORM_INPUT' and password = 'FORM_INPUT';
```

</details>

<details>
  <summary>Hint 4</summary>


The problem with SQL injection is that user input is not sanitized, and thus we can adjust the SQL query directly through the form input. Our text in the form ends up directly in the `FORM_INPUT` locations.


</details>


<details>
  <summary>Hint 5</summary>


So we know we need to adjust the query, and that we can use the form to adjust the query. First we need to 'break out' of the variable so we can not only adjust the user input but the entire SQL query itself. In SQL strings are surrounded by the `\`` `'` or `"` charachters. Input these characters in the email input and see what hapepns.

</details>

<details>
  <summary>Hint 6</summary>


Notice how `'` behaves differently and does not give a proper error instead it provides an error like `[object Object]`. This is an indication something is quite seriously wrong.

</details>

<details>
  <summary>Hint 7</summary>

So our goals to login would like something like this:
1. First we break out of string, so we can adjust the query.
2. Then we need to ensure that the query returns at least one row.
3. Then we need to ensure that the password portion of the query is ignored.

</details>

<details>
  <summary>Hint 8</summary>

1. Break out of string, with `'`
2. To return at least one row, we can utilize a condition that always returns true like `1=1`
3. With `--` we can apply comments to SQL, which we can utilize to ignore parts of the query.

</details>

<details>
  <summary>Hint 9</summary>

If you input `' or 1=1--` and any password you will authenticate the first entry in the `Users` table which coincidentally happens to be the administrator.
</details>


### Mitigation
Think on how you would mitigate this security vulnerability if you'd encounter it in your workplace. We'll discuss this at the end of the assignment.

### Optional
What other payloads can you think of that would allow you to login? Since you technically have full control of the query there are many!

#### Notes
As part of the Demo/Presentation explain how we get a list of users
- http://localhost:3000/rest/products/search?q=%27;
- https://pwning.owasp-juice.shop/appendix/solutions.html#order-the-christmas-special-offer-of-2014

## Lab 2: DOM XSS

How to spot, reflected in DOM
In Search
`<iframe src="javascript:alert(`xss`)">` 
Figure out the code that enables this
Danger of http://localhost:3000/#/search?q=bla (remote delivery)

## Lab 3: Persisted XSS attack
