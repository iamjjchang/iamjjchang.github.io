# Auth0 SAML Cloudflare Zero trust

1. Sign up Auth0 (https://auth0.com/signup)

![]({{ site.baseurl }}/assets/images/1.usermanagement.png)

2. Create User
   
![]({{ site.baseurl }}/assets/images/2.createuser.png)

3. Create an Application → Applications > Applications

![]({{ site.baseurl }}/assets/images/3.application.jpg)

4. Give it a name eg. Cloudflare Access and select **Single Page Web Application**

![]({{ site.baseurl }}/assets/images/4.cloudflareaccess.png)


Addon SAML2 Web App → Under this Application > Addons > **Select SAML2 WEB APP**

![]({{ site.baseurl }}/assets/images/5.SAML2.png)


Select **Settings** \
Under **Application Callback URL** input  https://<your-team-name>.cloudflareaccess.com/cdn-cgi/access/callback \ 
Scroll down and and click on **Enable**
<your-team-name> can be found under **Cloudflare Zero Trust > Setting > General Settings** 


![]({{ site.baseurl }}/assets/images/6.callback.png)


5. In this Addon screen, you can get SAML Configuration Parameters for Cloudflare ZT dashboard \
   1. Download Auth0 certificate & Metadata
   2. Take down of Issuer & Identity Provider Login URL

![]({{ site.baseurl }}/assets/images/7.addon.png)

# Configure Cloudflare Zero Trust dashboard

1. Login to your Zero Trust  Dashboard
2. Go to Settings > Authentication
3. On **Login methods** section, Click **Add new** > Click **SAML**

![]({{ site.baseurl }}/assets/images/8.addsaml.png)

4. In Email attribute name, fill in this 👇
    
[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress] 


<kbd> <img src="https://github.com/iamjjchang/Auth0-SAML-Cloudflare-Zerotrust/blob/master/9.optional.png" /> </kbd>

1. Once done click on test, you can see this result below

<kbd> <img src="https://github.com/iamjjchang/Auth0-SAML-Cloudflare-Zerotrust/raw/master/10.works.png" /> </kbd>

