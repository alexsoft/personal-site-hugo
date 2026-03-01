---
title: "Let's Get Encrypted"
date: 2016-03-01
draft: false
---

Long story short: I finally managed to get SSL certificate from "Let's Encrypt" and automate the process of certificates renewal.
And I would love to share my experience with you!

#### What is "Let's Encrypt"?

["Let's Encrypt"][letsencrypt] is the authority that provides free ssl certificates. This initiative is now supported by everyone,
so it means that you really get trusted certificates.
On December 3 they launched in public beta but unfortunately until yesterday I didn't have time to fully use it.

The history is that I had SSL certificate for this website but it was from Start SSL (they've changed design,
it's much-much better now) and to use certificate from there you need to go click couple of buttons on their website
and then concatenate two files in order to nginx to be able to use them correctly.
It was okay with me, but still I wanted to improve the whole proccess. And of course I was very interested in "Let's Encrypt" program.

The whole idea of "Let's Encrypt" is not only about free certificates but real security provided by certificates.
That's why they issue certificates only for 3 months. So there is less time of valid certificate in case it is compromised.

And of course it is highly recommended not to renew certificates manually. If you can automate it, do it!
So just configure your cron to renew certificates every month and that's it.

#### Technical details

Let's dive into technical details of this process. There is a console utility written in Python. You just clone it from github.

```
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./letsencrypt-auto --help
```

Next there are 4 ways to acquire your certificate:

##### Apache plugin
It is completely automated, it will do everything for you, but I don't use apache, and you should not either :)

##### Nginx plugin
It is not actually ready yet, so I even didn't try to use it

##### Standalone
You just run next command in your console and that's it.
```
letsencrypt certonly --standalone -d example.com -d www.example.com
```

No! Actually not really! Well, I mean it will be done but... The problem is that this approach needs to use port 80.
And you know what it means. You need to stop your webserver. And that's definitely not you would like to do
in your production environment.

##### Webroot
And here comes the last approach, and to be honest, for me it is the most preferred one.

```
letsencrypt certonly --webroot -w /var/www/example -d example.com
```

It validates your domain by accessing some random file in your webroot.
So ```-d``` parameter tells what domain you are getting certificate for.
And ```-w``` parameter tells the path to your webroot of your website.
And then it makes GET request to ```example.com/.well-known/acme-challenge/X``` where X is set of random symbols.
It creates ```/var/www/example/.well-known/acme-challenge/X``` for you and then tries to get it.
But I personally wouldn't like to have these directories and files in my webroot.
In fact you can specify any directory but then you need to configure webserver for it.

So, for example, if you would like to put those files in ```/var/www/ssl``` directory you should add a location
for your nginx server as such:

```
location ^~ /.well-known/ {
    root /var/www/ssl;
    try_files $uri =404;
}
```

By the way, you better place it before all other locations, just in case.

And then run

```
letsencrypt certonly --webroot -w /var/www/ssl -d example.com
```

After that you will get a congratulations message.

#### Nginx ssl configuration
Then, of course, we need to configure nginx to use these certificates.
After webroot and standalone approaches your certificate lies in ```/etc/letsencrypt/live/example.com```
In your nginx server context just add 3 rows.

```
ssl on;
ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
```

You might not need the ```ssl on;``` row if you specify ```listen 443 ssl;``` before.

```
sudo /etc/init.d/nginx reload
```

And it is done. Oh, no, we forgot about one last thing.

#### Automatic renewal
The thing is you need to setup crontab to launch the renewal process every month, or two, if you'd like.
Just add it to your crontab and be happy.

```
0 0 1 * * cd ~/letsencrypt && ./letsencrypt-auto certonly --webroot -w /var/www/ssl -d example.com
```

Sorry for not talking about apache configurations. I even don't remember when I used it. Anywhere.
Neither in my work place, nor for personal projects. Hope apache users are not offended.

Hope you enjoyed this article and it was useful for you!

[letsencrypt]: https://letsencrypt.org/
