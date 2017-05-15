---
layout: post
comments: true
title:  "The Internet and the HTTP Response Cycle"
date:   2016-05-13 14:05:01
categories:
---

In order to get a deeper understanding of what all the Internet is and the processes running in the background, it's always been helpful for me to jump back and look at the bigger picture (coding takeaways applicable to life in general!) - making sense of the bigger picture.  Here, I'll tackle the topic of the Internet and certain concepts and subject.
From a web developer's viewpoint, the Internet is a literal web, specifically of servers and users that exchange information through their network connection.  A web server is a software and/or hardware that stores the files and data of an application.  These files contain the HTML of the website and its assets, which would consist of CSS stylesheets  (styling), images, videos and other media, and Javascript files.
The server will either return a static page or a dynamic page.  A static page would be any file that maintains its state and sent back simply as is.  A dynamic page is one that may have to be compiled or generated using a database before it is sent to the requesting client.
Web servers wait for requests to be made so that they can send the bits and pieces of these files and the accompanying styling to the users requesting to view them. When a person types a URL, a web address, and presses enter, they are sending that request. For example, the site - http://www.example.com/index.html.
The information in the address consists of the protocol (http), the hostname (server) (www.example.com - amazon.com), and a file name (index.html).
The person is initiating the HTTP response cycle which is essentially the process of information being transferred from servers to user/client computers. That first action, of simply pressing enter, sends off an HTTP GET request that the server receives.  Upon receiving this request, the server looks through those stored files and if it finds one matching the URL address, it sends the file back to the client.
If on the client side the user is filling out a form and then pressing submit, they are making a POST request.  The main request verbs are GET, POST, PUT, and DELETE.  GET, as explained above, fetches an existing page and returns it as is to the client computer. POST, used when submitting a form, is creating a new resource.  With POST requests, data, via params, is carried back to the server to be saved.  PUT is used to update an existing resource, similar to a POST request it will also contain data to be updated and saved.  With PUT and POST requests the data being added and/or updated would be in the message body of the request, whereas a GET request won't have anything in the message body as everything that's needed is right in the URL - the address of the location of the file. DELETE requests simply delete an existing resource from a database.
When a file is not found, the server will still be required to return something.  In the cases where the file is not found, it will return an error message - the most typical being the "404 Not Found" message.  These are status codes that tell the client computer how to interpret the response it's receiving. Other common status codes are 2xx - anything following a 2 is successful. 200 is "OK", and if it's a GET request, the file will be in the message body. 3xx status codes redirect users when a file has been moved.

All of the covered data is sent via request and response messages, similar to the one below:

User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.com
Accept-Language: en, mi
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
ETag: "34aa387-d-1568eb00"
Accept-Ranges: bytes
Content-Length: 51
Vary: Accept-Encoding
Content-Type: text/plain
The message body would consist of the html content, ie:
<html>
   <body>

      <h1>Hello, World!</h1>

   </body>
</html>

We don't see the transfer of these messages or the details in the messages; we see what those messages produce.  To render web pages or update a profile page, the above is an important part of the background processes working to make that a reality.

(Network analyzers are tools you can install to see your own message headers and the numerous other details that messages contain).



Resources:
https://www.tutorialspoint.com/http/http_messages.htm

https://code.tutsplus.com/tutorials/http-the-protocol-every-web-developer-must-know-part-1--net-31177

https://github.com/learn-co-students/how-the-web-works-readme-re-coded-000
