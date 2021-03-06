# nginx.com favicon.ico fix
# Preventing PHP or other backend to be invoked when receiving requests for /favicon.ico

This may be serous problem when using WordPress because 404 errors usually are not cached - the entire WordPress mammoth will be invoked just to tell "there is no such file". This job is better to be delegated to a web server.

Method 1:

```
location = /favicon.ico {
	try_files $uri =404;
	log_not_found off;
	access_log off;
}
```

It first checks if there is /favicon.ico. This is waste of time, because most of the time there is no such file.

It also disables log messages to prevent clogging the logs.


Method 2:

```
location = /favicon.ico {
	return 404;
	log_not_found off;
	access_log off;
}

```

It works faster, because it does not look for favicon.ico on the disk.

I tried it with error code 204 instead of 404. But I noticed that nginx continues to execute other instructions instead of just give up and return the request. It did not add additional headers specified below in the nginx.conf if I change 204 to 404.

```
location = /favicon.ico {
	return 204;
	log_not_found off;
	access_log off;
}

add_header Testing-This-Blablabla 'blablabla';

```

```
$ curl -I  https://example.com/favicon.ico
HTTP/1.1 204 No Content
Server: nginx/1.10.2
Date: Mon, 30 Jan 2017 18:06:54 GMT
Connection: keep-alive
Testing-This-Blablabla: blablabla
```


When I change `204` to `404` it did not return the header:

```
location = /favicon.ico {
	return 404;
	log_not_found off;
	access_log off;
}

add_header Testing-This-Blablabla 'blablabla';

```


```
$ curl -I  https://example.com/favicon.ico
HTTP/1.1 404 Not Found
Server: nginx/1.10.2
Date: Mon, 30 Jan 2017 18:09:48 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
Vary: Accept-Encoding
```

If you don't have `robots.txt` file the problem is similar – every time when a robot (i.e. Google bot) is visiting your website, WordPress will be invoked just to say "error 404". The simplest and fastest solution is just to create empty `robots.txt` file.

