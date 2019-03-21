---
title: Setting up cassandra docker in Mac Part 2
date: "2015-08-14T22:12:03.284Z"
---

In part one I wrote an article about setting up Docker for Cassandra. Now Docker will act as a server so you still can't add your own _keyspace_ or doing any queries.

I really recommend using homebrew to make your life easier. You could install homebrew in <a href="http://mxcl.github.io/homebrew/">here</a>.

Setting cassandra on host
-------------------------

Now to install Cassandra on your host, run this command in terminal:

`brew install cassandra`

In case you don't have python installed, also run this command:

`brew install python`

And cql library for python:

`pip install cql`


Enter cqlsh on Docker
---------------------

Remember to start your docker first:

`docker-compose up`

To enter cqlsh:

`cqlsh [your docker ip]:[your cassandra port]`

Example:

`cqlsh 192.168.59.103:9042`

To know your docker ip on Mac:

`boot2docker ip`

Create keyspace on Cassandra
----------------------------

Keyspace is like database on traditional SQL. To create keyspace in Cassandra:

```
CREATE KEYSPACE demo WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor':1 };
```

To do queries on the keyspace you have just created:

`USE demo;`

Yup, that's it for the Cassandra introduction. Another great article you should read:
<a href="https://academy.datastax.com/demos/getting-started-apache-cassandra-and-python-part-i">https://academy.datastax.com/demos/getting-started-apache-cassandra-and-python-part-i</a>
