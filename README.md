## 📄 `README.md`

Hello FOSSEE Team, 

This repository tracks my step-by-step progress for the **DigitalOcean Server Setup with Keycloak SSO** internship screening task.

Before we move on I first want to thank **Thomas Stephen Lee** for his guidance on stages where I got stuck. His reference metarials really helped me tackle SSL errors, as a candidate it was a heartwelming experince of getting techinal support as promised by FOSSEE. 

Excluding this Readme, There are 5 more MD files that completely explains step by step process of how to completly build the project. 

+ [Server Setup README](https://github.com/CoolSrj06/fossee-system-adminstration-internship-task/blob/master/01-server-setup.md) 

+ [Keycloak Setup README](https://github.com/CoolSrj06/fossee-system-adminstration-internship-task/blob/master/02-keycloak-setup.md)

+ [Drupal Integration README](https://github.com/CoolSrj06/fossee-system-adminstration-internship-task/blob/master/03-drupal-integration.md) 

+ [Django Integration README](https://github.com/CoolSrj06/fossee-system-adminstration-internship-task/blob/master/04-django-integration.md) 
+ [PHP Integration README](https://github.com/CoolSrj06/fossee-system-adminstration-internship-task/blob/master/05-php-integration.md) 

# Current Status

First I want to demostrate the status of all the services running on droplet. I will be also attaching the Sceenshots as Proof at various stages to visually prove my implementation. 

## 1. Firewall status of the machine

![alt text](/images/image-13.png)

## 2. Service Status

![alt text](/images/image-14.png)

## 3. Keycloak Configuration

### 3.1 Drupal Client configuration Screenshot
![alt text](/images/image-15.png)
![alt text](/images/image-16.png)
![alt text](/images/image-17.png)

### 3.2 Django Client configuration Screenshot
![alt text](/images/image-18.png)
![alt text](/images/image-19.png)

### 3.3 PHP Client configuration Screenshot
![alt text](/images/image-20.png)
![alt text](/images/image-21.png)

# Demo

For a smooth demo of this project I have created a new user, on Keycloak.

+ **username:** fosseeAdmin
+ **password:** sysadmintask

## Demonstration

+ **Keycloak Link:** https://134.209.152.76:8443/
+ **Drupal Link:** http://134.209.152.76/ _(Since this was my first application to host, I rounted it to the ip address of the server, without creating a specific route, later when I realised my mistake it was too late to fix it, because change it would take me to fix serveral other configurations)_
+ **Djanko Link:** http://134.209.152.76/django/oidc/authenticate  _(This route redirect to the keycloak sigh in page
 Another route http://134.209.152.76/django/admin/login/?next=/django/admin/ , it take the user to the default django login page, which is for mannual login, it is not used in our scenario, so ignore it.)_
+ **PHP Link:** http://134.209.152.76/phpapp/login.php

_**Note:** All the links redirects users to the **Keycloak login page**._

![alt text](/images/image-22.png)

Now I will login via the username and the email I gave above, in Keycloak, and as expected I will be logged in all the other applications too.

![alt text](/images/image-23.png)

_It might be a situation in Drupal, that when you login in keycloak, You might not directly land on console page in Drupal, its not because you are not authenticated, its only because you have to tell Drupal now user is authenticated._

![alt text](/images/image-24.png)

_So just click on Log in with Keycloak_

![alt text](/images/image-25.png)

_And BOOM you will be logged in_

**Other application unlike Drupal are logged in automaticaly**

![alt text](/images/image-26.png)
![alt text](/images/image-27.png)

**In conclusion I would say this is the complete and simple demonstration of my project.**

### Concluding Keynotes

+ **SLO:** Unlike SSO, SLO (Single Logout) is not configured for all the applications, if you logout from Drupal you will be logged out from Keycloak (vice-versa), but same will not happen for now, but it can be done in later stages.

+ **User registration:** For a new user registration, for time being the user can be created by the admin only with giving him all the permissions. 

## Final Note:

I have invested a lot of my time and effort in completing this task. This project has given me a lot of learning and I have also understood the ajenda of the project I will be working on if I get selected in the internship. 

I got stuck at many places on of the recent mistake I did was that I turned on **verify Email** option in my realm settings but had never set up SMTP. After logging out, I couldn’t log back in because Keycloak forced me to verify my email… but no email was being sent. Basically, I bricked my admin account.

At first, I tried the Keycloak CLI (kcadm.sh), but it also refused to log me in because my account wasn’t “fully set up.” Then I dug deeper:

🔹 Found out my Keycloak was still running on the embedded H2 DB (not MariaDB).
🔹 The fix was to use the export/import feature:

Exported the master realm JSON.

Patched it with **jq** to set:

**"verifyEmail": false** at the realm level

**"emailVerified": true** for my user

cleared **"requiredActions"**

Imported the patched realm back with **kc.sh import --override true**.

_After restarting Keycloak, I was finally able to log in again 🎉._
The lessons I learned:

    ✅ Always configure SMTP before enabling Verify Email.
    ✅ Don’t rely on the default H2 DB in production — use MariaDB/Postgres.    
    ✅ Keep a bootstrap admin user (KEYCLOAK_ADMIN) handy for emergencies.

**Thanking You
Srijan Maurya
Final Year Student at VIT Bhopal University**



