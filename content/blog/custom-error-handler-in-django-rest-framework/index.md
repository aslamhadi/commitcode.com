---
title: Custom error handler in Django Rest Framework
date: "2017-02-17T22:12:03.284Z"
---

Django personally is still my favorite framework for backend. It provides great documentation, active community, and it's provides great API for us developers to get started really quick. This goes to Django Rest Framework as well.

What I love from Django Rest Framework (DRF) is they provide great error handler and if you want to customize your error handler, you can do it easily. Below is the default error from DRF, it shows the field and list of error for that field.

```
 {
  "name": [
    "Bidang ini tidak boleh kosong."
  ],
  "unit_id": [
    "Bidang ini tidak boleh kosong."
  ]
}
```

I got a requirement which shows the http code in the response, and also shows all errors in `errors` property. This way the FE can grab the error list with `response.errors`. This is what the FE team wants:

```
{
  "status_code": 400,
  "errors": [
    "unit_id : Bidang ini tidak boleh kosong.",
    "name : Bidang ini tidak boleh kosong."
  ]
}
```

### Implementation

This is how you do it. First get the standard error handler from DRF. If it contains exceptions, loop all the errors and append those errors into one list. Complete gist below:

<script src="https://gist.github.com/aslamhadi/682e7764e240ad932223c7f89861b924.js"></script>

After that, make sure to update the DRF in settings:

```
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'my_project.my_app.utils.custom_exception_handler'
}
```
