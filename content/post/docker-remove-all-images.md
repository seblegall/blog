---
title:  "Docker : Remove all images"
description: "How to remove all images"
date:   2016-06-11T19:49:34+02:00
author: Sébastien Le Gall
categories: [docker]
---
If you have ever add to work with docker and and docker-compose, you probably often have mistaken in you Dockerfile. Specially when you work with multiple container build from multiple Dockerfile and tag differently.

Here is a simple way to clean all your docker images.

Show images :
{% highlight sh %}
$ docker images
{% endhighlight %}

Delete all container :
{% highlight sh %}
$ docker rm $(docker ps -a -q)
{% endhighlight %}

Delete all images :
{% highlight sh %}
docker rmi $(docker images -q)
{% endhighlight %}
