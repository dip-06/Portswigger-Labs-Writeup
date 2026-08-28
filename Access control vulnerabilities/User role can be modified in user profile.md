# User Role Can Be Modified in User Profile

**Lab:** Improper access control allowing a user to modify their `roleid` and gain unauthorized access to the admin panel.
**Category:** Access Control
**Difficulty:** Apprentice

![](Screenshots/Screenshot_20260828_112449.png)

To solve this lab we have to gain admin access and delete the user `carlos` . Given that the user with `roeid : 2`  can access the admin panel.
Open the website in burp's browser and login using given credentials `wiener:peter` .
Now turn your intercept on and change the emailid :
![](Screenshots/Screenshot_20260828_113614.png)

Click on update email and Intercept the request.
Now in the `POST` request add another `key:value` parameter `roleid:2` :
 
![](Screenshots/Screenshot_20260828_112246.png)

Now send the request. Then you will get admin panel option in your account page :

![](Screenshots/Screenshot_20260828_112339.png)

Go to admin panel and delete the user `carlos` , And the lab is solved.
