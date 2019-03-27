---
title: Background Process In Scala
date: "2017-02-11T22:12:03.284Z"
---

So yesterday I was hit by an intermittent bug in the application. The case is like this. Imagine you're a book store owner and every time you add a new book, the application will:

<strong>Frontend side</strong>

* Upload the cover book image to Amazon S3
* Get image path from S3
* Send the path alongside the other book's information to backend application

<strong>Backend side</strong>

* Get the image path and other book's information from frontend
* Validate the data
* After validated, store the information to database
* Send the image path 3rd party API (let's call it Mawar - not real name) where it will extract more information

We will be more focusing on the backend side for now since it uses Scala, it's an awesome language btw you should check it out. Now, if you translate it to Scala, we will get more or less like this one:

```
def confirmation = Action.async(parse.json) { request =>
  Validated[AddBookInputModel](request, AddBookInputModel.formatter)(executor) { model =>
    for {
      ... (some additional process)
      book <- bookService.addBook(model.book)
      processMawar <- mawarService.processMawar(model.imageUuid)
    } yield {
      ... (serve response to frontend)
    } 
  } 
}
```

This code works well (at the first time), so it got promoted to staging environment. And then something happened. They say sometimes they get intermittent bug when adding a new book.

![noo](./noo.png)

After we got that information, we get back to analyzing the code. If we look at the process both at frontend and backend side, there are 3 expensive processes:

* Uploading image cover to S3
* Store book information to database
* Call Mawar microservice to process the image 

In this case, since I didn't change the image uploading process, it downs to two suspects ;)

### Sequential code in Scala

As we now that in Scala, every line in `for yield` loop which using `<-` will be process sequencially. Suppose you want to create a cup of coffee, this is basically what you do:

* Grind the coffee
* Heat the water
* Brew the coffee with water
* Add some Milo (just because you can, why not)

If we translate this to Scala code, we basically have this:

```
// going through these steps sequentially:
def prepareCappuccino(): Try[Cappuccino] = for {
  ground <- Try(grind("arabica beans"))
  water <- Try(heatWater(Water(25)))
  espresso <- Try(brew(ground, water))
  milo <- Try(addMilo("milo"))
} yield combine(espresso, milo)
```

These processes will be done one by one, you can not brew the coffee if you don't have hot water ready. And you can't add Milo if the coffee is not ready (well actually you can in real life,, but,, yeah). The point is, you are blocked into each process.

### Back to Scala code

So in our case, the backend can not serve the response if it doesn't get response from Mawar API. And because information from Mawar won't get used directly by the user, we don't need to wait for Mawar API to finish the process. 

How do we fix this? We change two parts of the code. The first one by updating the `for yield` loop. We change `<-` to `=`. By changing this, we no longer wait the result of Mawar API and just serve the response to the user. It become background process. So the code become like this:

```
def confirmation = Action.async(parse.json) { request =>
  Validated[AddBookInputModel](request, AddBookInputModel.formatter)(executor) { model =>
    for {
      ... (some additional process)
      book <- bookService.addBook(model.book)
      processMawar = mawarService.processMawar(model.imageUuid)
    } yield {
      ... (serve response to frontend)
    } 
  } 
}
```

The second change is how we call the Mawar API. We add timeout so if the process took longer than the specified time, the system will stop the thread for the process. It's really helping to prevent the User getting timeout.

```
ws.url(url).withRequestTimeout(5000).get()
```

### References

* <a href="https://www.playframework.com/documentation/2.4.x/ScalaWS#request-with-timeout" target="_blank">Play framework documentation</a>
* <a href="http://danielwestheide.com/blog/2013/01/09/the-neophytes-guide-to-scala-part-8-welcome-to-the-future.html" target="_blank">The Neophyte's Guide to Scala (recommended!)</a>
