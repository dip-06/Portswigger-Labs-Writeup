# User Role Controlled by Request Parameter

**Lab:** Broken access control caused by a forgeable role parameter, allowing a user to modify their apparent role and gain unauthorized administrative access.
**Category:** Access Control
**Difficulty:** Apprentice

![](/Screenshots/Screenshot_20260828_112022.png)

To solve this lab we have to login as the `admin` and delete the user `carlos` .
First we open the lab and login with the given credentials `wiener:peter` .
Now turn the intercept on and reload the page. 
In the request you will see a parameter `Admin=false` in cookie. Now change it to `true` .Then send the request.
For all the request do the same.

![](/Screenshots/Screenshot_20260828_111734.png)

Now you will see you got a admin panel option in the page. Click on the admin panel and again set `Admin=true` . Then you will get the admin panel access and delete the user `carlos` .

![](/Screenshots/Screenshot_20260828_111929.png)

Now delete the user `carlos` and again set `Admin=true`  . Then the user will be deleted and the lab will be solved.
