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


  The vulnerability is located at the [login page]](http://localhost:3000/#/login).


</details>

<details>
  <summary>Hint 3</summary>


  If you are familiar with SQL, think of what a login query generally looks like. If you are not have a look at hint 3!


</details>

<details>


  <summary>Hint 4</summary>
  The login query looks something like the query below and will log the user in if there is a match 

    ```SQL 
    SELECT * from users where email = 'FORM_INPUT' and password = 'FORM_INPUT';
    ```


</details>

<details>
  <summary>Hint 5</summary>


The problem with SQL injection is that user input is not sanitized, and thus we can adjust the SQL query directly through the form input. Our text in the form ends up directly in the `FORM_INPUT` locations.


</details>


<details>
  <summary>Hint 6</summary>


So we know we need to adjust the query, and that we can use the form to adjust the query. First we need to 'break out' of the variable so we can not only adjust the user input but the entire SQL query itself. In SQL strings are surrounded by the `\`` `'` or `"` charachters. Input these characters in the email input and see what hapepns.

</details>

<details>
  <summary>Hint 7</summary>


Notice how `'` behaves differently and does not give a proper error instead it provides an error like `[object Object]`. This is an indication something is quite seriously wrong.

</details>

<details>
  <summary>Hint 8</summary>

So our goals to login would like something like this:
1. First we break out of string, so we can adjust the query.
2. Then we need to ensure that the query returns at least one row.
3. Then we need to ensure that the password portion of the query is ignored.

</details>

<details>
  <summary>Hint 9</summary>

1. Break out of string, with `'`
2. To return at least one row, we can utilize a condition that always returns true like `1=1`
3. With `--` we can apply comments to SQL, which we can utilize to ignore parts of the query.

</details>

<details>
  <summary>Hint 10: Final Solution</summary>

If you input `' or 1=1--` and any password you will authenticate the first entry in the `Users` table which coincidentally happens to be the administrator.
</details>


### Mitigation
Think on how you would mitigate this security vulnerability if you'd encounter it in your workplace. We'll discuss this at the end of the assignment.

### Optional
What other payloads can you think of that would allow you to login? Since you technically have full control of the query there are many!

#### Presenter Notes
As part of the Demo/Presentation explain how we get a list of users
- http://localhost:3000/rest/products/search?q=%27;
- https://pwning.owasp-juice.shop/appendix/solutions.html#order-the-christmas-special-offer-of-2014

## Lab 2: DOM XSS
### Assignment
Somewhere one the site we can perform a DOM XSS attack meaning we can execute Javascript.

<details>
  <summary>Hint 1</summary>


  The exploit is located in the search functionality.


</details>

<details>
  <summary>Hint 2</summary>


  The exploit allows you to use HTML.


</details>

<details>
  <summary>Hint 3</summary>


For example try: `<b style="color:red;">WOW, BIG MISTAKE</b>`. Can you imagine the consequences of this?


</details>

<details>
  <summary>Hint 4 Final Solution</summary>


  For example use `<iframe src="javascript:alert(`xss`)">` in the search field. Also notice that we can use this delivery remotely since the search query is in the page URL: `http://localhost:3000/#/search?q=%3Ciframe%20src%3D%22javascript:alert(%60xss%60)%22%3E`. We just need to encode the URL.


</details>

### Mitigation
Think on how you would mitigate this security vulnerability if you'd encounter it in your workplace. We'll discuss this at the end of the assignment.

### Optional
If you are familiar with frontend try to make the website display a 'YOU HAVE WON' banner where the search result shows. You can then add a malicious URL for the user to navigate to. 

#### Presenter Notes
As part of the Demo/Presentation explain how we get a list of users
- `http://localhost:3000/rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users--`
- Detailed explanation [here](https://pwning.owasp-juice.shop/appendix/solutions.html#order-the-christmas-special-offer-of-2014).

## Lab 3: Persisted XSS attack
For the next lab we will use multiple exploits and configuration mistakes together to attack the users of the website. Our persisted XSS attack utilize a vulnerability in a javascript library used by the application. The prerequisites steps below explain how we would find out that the library is used. 

<details>
  <summary>Prerequisites</summary>

  1. There is a serious misconfiguration on the webserver of this site, ftp access is available on `http://localhost:3000/ftp` on Mac and `http://192.168.99.100:3000` on Windows.
  2. Notice that a file exists with the name of `package.json.bak` For those unfamiliar a `package.json` file is a file used in frontend applications to document which versions need to be installed of dependencies.
  3. Downloading this file will throw an error because of some poorly implemented security measures by the FTP server which should only allow `pdf` and `md` files to be downloaded. 
  4. So let's bypass that download filter with a Poison Null Byte, use the URL `http://localhost:3000/ftp/package.json.bak%2500.md` to download the file anyway tricking the FTP server into thinking you download a MD file.
  5. We now have the `package.json` file downloaded and have all exact version numbers that the frontend will use.
  6. We can then use this list to make an inventory of all vulnerabilities in the frontend libraries.


</details>

We utilized the steps described in the prerequisites step to find out that the site uses a weak version of sanitize-html: `"sanitize-html": "1.4.2"`. 

The security report of this vulnerability reads the following:
```
Sanitization is not applied recursively, leading to a vulnerability to certain masking attacks. Example:

I am not harmless: <<img src="csrf-attack"/>img src="csrf-attack"/> is sanitized to I am not harmless: <img src="csrf-attack"/>

Mitigation: Run sanitization recursively until the input html matches the output html.
```

### Assignment
Armed with the above information are you able to perform a persisted XSS Attack?

<details>
  <summary>Hint 1</summary>


The exploit is located on a place where you can add content for other users to see. Sadly on the latest version of the docker container this exploit might not work. But it's worth running through the hints to get a basic understanding of how a vulnerable library can have consequences for all users of your site.


</details>

<details>
  <summary>Hint 2</summary>


You submit feedback on one place and it is visable on another place. Can you find such a usecase on the site?


</details>

<details>
  <summary>Hint 3</summary>


The exploit is located at the Customer Feedback (`http://localhost:3000/#/contact`) page. The content you post will be available here in the carousel `http://localhost:3000/#/about`.


</details>

<details>
  <summary>Hint 3</summary>


Reading the security report it becomes clear that by using `<` in the comment field we can execute HTML again.

</details>

<details>
  <summary>Hint 4</summary>


Try a payload of `<<script>Foo</script>iframe src="javascript:alert('xss')">`.

</details>


### Mitigation
Think on how you would mitigate this security vulnerability if you'd encounter it in your workplace. We'll discuss this at the end of the assignment.

## Lab 3: Redirect a user to your malicious site
The Juice Shop has quite a few redirects to social media pages and such, we can utilize these to redirect to our own malicious site. We can then trick the user into clicking on these links and redirect them to our own site. 

### Assignment
Find out a way to redirect the user to another site.

<details>
  <summary>Hint 1</summary>


Inspect the links of the site and try to find a link where a redirect logic is utilized. 

</details>

<details>
  <summary>Hint 2</summary>


The link in question is the GitHub link.

</details>

<details>
  <summary>Hint 3</summary>


The link in question is the GitHub link.

</details>

<details>
  <summary>Hint 4</summary>


The link in question is the GitHub link.

</details>

<details>
  <summary>Hint 5</summary>

The link to utilize is `http://localhost:3000/redirect?to=https://github.com/bkimminich/juice-shop`

</details>

<details>
  <summary>Hint 6</summary>

Notice that with a payload of `http://localhost:3000/redirect?to=https://www.google.nl` we will get an error. Nice security! Or maybe not can you get around this?

</details>

<details>
  <summary>Hint 7</summary>

Obviously the developers of the app were smart enough to implement a whitelist and `https://github.com/bkimminich/juice-shop` is on that whitelist. Maybe the whitelist was implemented poorly we can use this to our advantage?
</details>

<details>
  <summary>Hint 8</summary>

We can modify URLs in a way that has no impact to the actual URL and incorporate the original whitelisted URL into our own malicious URL. For example by adding a `GET` parameter by doing something like this `https://www.google.nl?someValue=BLA`. Can we use this to add the original URL back into our malicious site URL?
</details>

<details>
  <summary>Hint 9</summary>

We can craft a redirect URL like `https://www.google.nl?bla=https://github.com/bkimminich/juice-shop` in order to still have the original URL in the message, since the redirect validation does not look at the entire input but accepts partial matches this is enough to bypass validation. We can now use the official URL to redirect to our malicious site. 
</details>

<details>
  <summary>Hint 10: Final Solution</summary>

The final URL is: `http://localhost:3000/redirect?to=https://www.google.nl?bla=https://github.com/bkimminich/juice-shop`


</details>

### Mitigation
Think on how you would mitigate this security vulnerability if you'd encounter it in your workplace. We'll discuss this at the end of the assignment.
