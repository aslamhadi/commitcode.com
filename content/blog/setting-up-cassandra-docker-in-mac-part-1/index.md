---
title: Setting up cassandra docker in Mac Part 1
date: "2015-08-11T22:12:03.284Z"
---

I see that there is still not enough writings about docker and cassandra in Mac so I thought I should write one.

**Setting up the Docker File (I'm using flask)**

	FROM python:2-onbuild
	RUN pip install -r requirements.txt

**Now setting up docker-compose.yml**

	olarkCassandra:
	  image: abh1nav/cassandra
	  volumes:
	    - /data/cassandra:/opt/cassandra/data
	  ports:
	    - 9042:9042
	    - 9160:9160

Now that you have configured the docker, it's time to build it.

Run `docker-compose build` and `docker-compose up`

And,, that's it. In the next article, I'll write about how to execute query on Cassandra :)
