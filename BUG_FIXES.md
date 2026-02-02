                                    BUG FIXES(UNOLO)

1.	So, first time I enter the login credentials even the right ones, I am thrown with login failed error, 

I open the dev tools -> network tab, select the fetch tab  look for auth/api error code  says 500, so basically that means that the request reached the backend but my backend code threw an exception.

SO, I NEED TO DEBUG THE BACKEND CODE FOR LOGIN AUTHORIZATION FIRST.

#BUG 1: 

Login fails with the error status 500 even with the correct status
API returns: 500
Errorcode(backend): secretOrPrivate key must have a value: jwt.sign(payload,SECRET);
Location: Json/webtoken/sign.js

1.	First I check for the secret key inside the .env file, and see if the secret key is added there.In my case, backend has a  .env.example folder but no .env folder, so I had to copy everything from example to .env after creating it.(Creating a new .env folder fixed my secret undefined 500 error because now my jwt.sign() has a secret key.)

2.	Then I checked for the dotenv, if it is loaded or not. It needs to be loaded before all the routes are loaded, so I import it first at the top.

#BUG 2:

Another important minor issue is that user.password is inside the jwt which can be fatal because the jwt is stored in local storage, and so anyone with a token can easily encrypt it, its not a safe practice.
1.	So, I just need to remove the password from the jwt and that’s it.
2.	Now JWT only has name,email,id and these are all non-sensitive data as they should be, jwt should only contain non-sensitive data so that clients can’t have access to sensitive data like password.

 
#BUG 3:

On successfully logging in to the dashboard, I open the dev tool and inside the network tab, I see a 304 error code: which basically says that it’s a browser cache response error, browser already has the response as cache due to which I don’t see any json and can’t tell what the API is actually returning.
So, I need to disable the cache.

#BUG 4:

So, after checking at the code 200, I open the json response that its getting from the  backend, and in the response I saw details like team size, team members and their details, then I saw active check ins as 1 but today_checkins as empty which doesn’t make any sense as they both cannot be true together.
1.Backend finds the active check ins but it is failing to identify or mark those check ins as today_checkins.

2. Once I look through the dashboard.js, I can see that today’s date is stored using  data() and ISO timestamp but SQLlite parses date in UTC so there is a mismatch. I try to get rid of the date( ) method. 
Date() derives local time from system but sqlLite parses date in the UTC format thus causing the mismatch.

3.	Most importantly active checkins was counting all the rows of checkins, so it may even count the duplicate check ins that may be created by a single employee with multiple records.
So, I had to use the DISTINCT ch.employeeID to count only the distinct checkins not the duplicate entries by single employee.

#BUG 5:

History page not loading
This bug is actually really obvious.
In the network tab, we can see that it says 0/23 requests, so basically we can say that frontend is not calling any of the History API at all.
So, I looked for simple bugs like whether useEffect was called, or methods were even loaded, but there were no such errors or any syntax errors.

1.	So, first problem is that checkin.reduce() is being called but no data is loaded as of such so it returns will a null value as in useState, and react  runtime crashes.
2.	So, I just [ ] because for representing an empty list, I needed the [ ], not null.
Null is non iterable, thus it crashed when reduce( ) was called on it.

#BUG 6:

Employee Dashboard is not loading:
So, I know that much that it is a frontend issue related to failure of API calls, most probably the employee to dashboard is failing.
API return : 500
When I open the dev tools and network tab, I can see a 500 error code, and when I look into the response I see a json message: {failed to fetch dashboard}, so it confirms one thing that this is a backend failure because it’s a catch block error message.
1.	This is a database syntax issue, the code is valid for MySql but we have used SQLlite, so it needs to replaced with sqllite syntax.
2.	So I just replaced the  DATE_SUB(NOW(), INTERVAL 7 DAY) which used MySql
with the datetime(‘now’, ‘-7 days’) that was syntactically correct for SQLlite.

So after inspecting the 500 error code on network tab, I found the bug origin and went through the dashboard for employee in the backend and found out about the wrong syntax and replaced it with the correct syntax.

#BUG 7:
(Implementation of Feature A).
Distance Calculation:
After going through the checkin backend code, I found out that there was no way of calculating the distance, So I needed to calculate the distance as well and insert that distance into distance_from_client log.
I calculated the distance using the Haversine formula(ref: google) after fetching the client location.


#BUG 8:
Check_Out fail message:
So after clicking on the checkout message, I was shown a alert that the checkout was failed, then again after I reviewed the checkout endpoint in the checkin file, I found out that again the issue was similar to the history load page, here also DATE( )  method was used.
1. So, I replaced the DATE( ) method which is valid for MySQL  with the datetime(“now”) which is a valid syntax for the SQLlite inside the 
UPDATE( ) query.
2. Now, saving it and restarting the backend server led to a successful checkout.



