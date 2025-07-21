## 💡 Cdnio

### Introduction of this task
  This was a web challenge that attacked logical vulnerabilities and information leakage. It entailed finding the endpoints of the web application, identifying vulnerabilities, and finally getting the flag.

### 🛠️ Process of solving Cdnio

1. I download all the files and got a host name of the target machine (94.237.61.242:52644).

   Therefore, i use the URL :  http://94.237.61.242:52644  and tried to open the website.

   ![Screenshot showing GET flag result](./image/Cdnio_open.jpg)

   ✅ Why I did this:

   This is where the challenge started. For most of Hack The Box web challenges, it is necessary to get access to the provided hostname in order to start the enumeration process. I browsed the target so that I could know what kind of web application I was dealing whether it supported login or register functionality, had exposed endpoints, or public API-like behavior.


   🔍 What i inferred:
   
   By loading the page in the browser, I noticed that there was a login feature and a registration feature. This told me that the site likely worked with user roles or user data. It suggested the possibility of testing for logic flaws, authentication bypass, or information leakage. It also suggested that part of the challenge would be talking to an API after one is logged in, because the professional look of the interface and potential backend integration suggested that.

3. Before i register and log in, i opened the Burp Suite to intercept all HTTP request and knowining how the website handles user.

   ```bash
   
   java -jar burpsuite_community_v2024.1.1.jar
   
   ```

   
   ✅ Why I did this:
   
   Burp Suite lets me see and analyze browser-to-target-web-application network traffic. It allows me to detect unseen endpoints, request structures, and vulnerabilities like broken authentication or insecure data exposure.
   
   🔍 What i inferred:
   

   Intercepting the register and login requests, I was able to analyze the request headers, body, and response data which is very useful in identifying authentication mechanisms, tokens, or wrongly exposed user data.


3. After i open the website successfully , i can see this is a webpage can search the username and password.

   There is also a register bottom on the top of the website.

   Therefore, i go to register and register randomly and it shows that i registered successfully.

   ![Screenshot showing GET flag result](./image/Cdnio_register.jpg)


   ✅ Why I did this:

   I needed to sign up as a user in order to have access to the entire site functionality. Most web challenges include user authentication, API data exposure, or access control bugs, so creating an account is one common initial step to get some context.

   🔍 What i inferred:

   Because the page allows for searching user credentials, my initial assumption was that this functionality was the crux of the challenge. By setting up an account of my own, I could test out how the search operates, whether it leaks sensitive data, and if it might be used in a malicious manner to search other users' information.

4. I can see register successfully when i register my username and the password.

   After i resgitser, i log in into the search system to see how it works.

   ![Screenshot showing GET flag result](./image/Cdnio_profile.jpg)

   The image shows that after i use my username and password and click search, it shows a profile of my username, password, email, API key and the create date.

   
   ✅ Why I did this:
   
   I wanted to see how the application handles user data and what is returned when I submit my own login credentials as a search box.

   
   🔍 What i inferred:
   
   The system performs a lookup based on the username and password entered, and then just returns sensitive account data , the API key in the frontend response.
This move is an indication of a potential information disclosure vulnerability and implies that if we were able to predict or find another correct username or password combination, then we might be able to access another user's API key or secure data.

5. I go back to the Burp Suite to see any request when i am doing register, log in and search.

   ![Screenshot showing GET flag result](./image/Cdnio_api.jpg)

   We can see after i register there is a POST request. Therefore, i sent this POST request (200) to the repeater.

   ✅ Why I did this:
   
   POST requests typically carry sensitive data like username and password. Analyzing these requests, I may have a clearer understanding of how registration and authentication are handled by the backend, and check for any vulnerabilities.

   🔍 What i inferred:
   
   By intercepting and examining these requests, I could identify potential injection points, understand how the JWT token is retrieved, and see how sensitive information is being handled. This is a very crucial step prior to executing any other attack such as token replay or privilege escalation.
   
6. When I logged in, I noticed the site recalled that I was there and showed my personal information.
   
   But I wasn't catching any cookies being sent out in Burp Suite, so I questioned whether the site perhaps was authenticating with a JWT token instead.
   
   JWT tokens usually have to be cached in the browser's sessionStorage or localStorage.

   Therefore, i open the page website and open the developer tools, find Local Stroage/ Session Storage at the Application.

   In the Local Storage, i found a JWT token.

  Neither the server was making use of conventional cookie-based sessions.
• The presence of user-specific data after logging in hinted at a token-based system.
• JWTs are often stored in localStorage, so this was the most likely place to look.
→ This helped me in obtaining the JWT and scheduling the next steps to be performed on the API.

   The token is:
   
  ```bash

  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhZG1pbjEyMyIsImlhdCI6MTc1MjYzOTY5NSwiZXhwIjoxNzUyNzI2MDk1fQ.8d7BPDxypJ6JRA6Pzph7H92kV79MV_02v5HdBJKHXg0
  ```

  ✅ Why I did this:

  I thought the site was making use of token-based authentication (JWT) instead of cookies, I looked for any authentication tokens in the browser's local storage.

  🔍 What i inferred:

  - Neither the server was making use of conventional cookie-based sessions.

  - The presence of user-specific data after logging in hinted at a token-based system.

  - JWTs are often stored in localStorage, so this was the most likely place to look.
      
  This helped me in obtaining the JWT and scheduling the next steps to be performed on the API.
      
7.  I got the token , and this token can let me pretent a user to send the request.

    Therefore, i use 'curl' to sen a POST request.

    ![Screenshot showing GET flag result](./image/Cdnio_token.jpg)

    ✅ Why I did this:
    
    Noticed that once I had logged in, the site was showing my user profile, yet I hadn't noticed any cookies being used for authentication within Burp Suite.
   
    This led me to believe that the site might be using a JWT token instead of cookies.

    JWTs are usually stored in localStorage or sessionStorage within the browser, so I opened the Developer Tools and found the token in Local Storage.

    To see if the token could be used to retrieve protected resources, I crafted a curl request to the /visit endpoint and included the token in the Authorization header.

    🔍 What i inferred:

    The endpoint perhaps returns different data depending on the rights of the user.
    
    My next step, then, was to try to access profile.png directly using the same host and port to see if it would give back any sensitive data, such as the flag.

9. I used JWT token to call .visit this endpoint and the result shows that {"message": "Visiting URI: profile.png"}.

   Therefore , i tried to use curl to catch this file.

   I did not use JWT token because i guess from the last step , i have the right to access to the file.

   ![Screenshot showing GET flag result](./image/Cdnio_information.jpg)

   ✅ Why I did this:
   
   As the server had given back the filename profile.png, I inferred that the file would likely be stored on the server and would be directly accessible. This is a common misconfiguration — making an encrypted file appear like a public file.
   
   🔍 What i inferred:

   The /visit endpoint likely triggered the creation or revelation of profile.png.
   
   - The file path was anticipatable and not protected.
   
   - These static files are most often publicly available without authentication.

11. After i curl to catch the file, i got the flag.

    Flag: HTB{cDN_10_OoOoOoO_Sc1_F1_iOOOO0000}

     ![Screenshot showing GET flag result](./image/Cdnio_getFlag.jpg)


### 📕 Summary of this task 

  In this challenge, I looked at a web challenge that consisted of user authentication and API access. I began by registering using a random username and password, then observed how the site reflected personal profile details through its search feature. Using Burp Suite to inspect the network traffic, I discovered that the site uses a JWT token for authentication, which is stored in the browser's Local Storage.

  Later, I discovered an endpoint named .visit, which appeared to give access to some server-side files. I attempted to access the file directly using curl, and to my surprise, I was able to obtain a JSON response containing the flag.


### 📍 Challenge Faced
  - Obscure authentication mechanism at first: The website maintained my session after logging in, but there were no cookies intercepted by Burp Suite. This led me to suspect JWT tokens, where I then found they were being stored in Local Storage.

  - I tried accessing without passing the JWT token, and it still worked. This revealed a critical server-side vulnerability where access control was missing or misconfigured, allowing unauthenticated access to secured resources.

