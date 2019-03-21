---
title: Data Volumes using Docker Compose
data: "2015-09-01T22:12:03.284Z"
---

If you are developing a web app using Docker, you will change your code so many times. The problem with developing using Docker, your files are build in the first time. So you need to stop the Docker, run `docker-compose build` and then `docker-compose up` to test your app again. This problem can be solved using Data Volumes.

Data volumes features:
* Volumes are initialized when a container is created. If the container’s base image contains data at the specified mount point, that existing data is copied into the new volume upon volume initialization.
* Data volumes can be shared and reused among containers.
* Changes to a data volume are made directly.
* Changes to a data volume will not be included when you update an image.
* Data volumes persist even if the container itself is deleted.


How to add data volumes in Docker Compose
-----------------------------------------

Tag to create data volumes in docker compose is volumes. This is an example of using data volumes in Flask:

	web:
	  build: .
	  command: python application.py
	  volumes:
	    - .:/usr/src/app
	  links:
	    - hadoop
	  ports:
	    - 9000:9000

In the example above I set the volume from the code's folder (root) to `/usr/src/app` in python container. This means when you are updating your code, the docker will automatically detect the changes.


How to share files between containers
-------------------------------------

Imagine if you have files in web containers above and hadoop container, how does hadoop knows where to get the files? We can set it up using data volumes as well :)


	web:
	  build: .
	  command: python application.py
	  volumes:
	    - .:/usr/src/app
	  links:
	    - hadoop
	  ports:
	    - 9000:9000

	hadoop:
	  image: labianchin/hadoop
	  volumes:
	    - .:/home
	  ports:
	    - 50070:50070
	    - 50075:50075
	    - 10000:10000


In the example above we can share the files between web container and hadoop container. Just access from /home in hadoop and you will find your files :)
