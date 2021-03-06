Easy-peasy record creation via Codelets
=======================================

This [Codelet](../codelets.md) example will show you how to create a
super-simple HTML form to create new rows in your Fieldbook database.

We're going to start with a simple sheet that just tracks people who have
seen this example.  Check out [this
book](https://fieldbook.com/books/56cccbd72ba55103004f278d).  It just has
Name and Country.  Feel free to copy that book and play with it yourself!

Here is a screenshot:

![people-sheet](../images/codelet-form-sheet.png)

We are going to create a small codelet that, when retrieved, renders an HTML form,
and on POST, creates a record.

Code
----

First, let's create the endpoint function that will dispatch based on the HTTP
method (is this request getting the form to display, or is it submitting data?)

```javascript
exports.endpoint = function (req, res) {
  if (req.method === 'GET') {
    return renderForm(req, res);
  } else if (req.method === 'POST') {
    return postForm(req, res);
  } else {
    throw new Error('Bad method: ' + req.method);
  }
}
```

Then we need to render the form, which just requires setting the
Content-Type and returning a string (see the
[codelets response documentation](../codelets.md#codelet-responses) for more
info).

```javascript
var renderForm = function (req, res) {
  res.type('text/html')
  return `
    <head><link rel="stylesheet" href="//yegor256.github.io/tacit/tacit.min.css"/></head>
    <body>
      <form method="post">
        Name: <input name="name" type="text">
        Country: <input name="country" type="text">
        <input type="submit">
      </form>
    </body>
  `;
}
```

Here we are using the cool little [Tacit CSS](//yegor256.github.io/tacit/)
library, and also ES6's [template strings](//developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

And finally, we need to create the record from the resulting post:

```javascript
var Q = require('q');
var postForm = Q.async(function * (req, res) {
  yield client.create('people', req.params);
  return 'Thanks!'
})
```

The `client` object is a pre-initialized JavaScript object for talking to
Fieldbook's REST API.  It is an instance of the
[fieldbook-client](//github.com/fieldbook/fieldbook-client) module.  We are
also using
[generators](//developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
from ES6, as well as Q's
[async](//github.com/kriskowal/q/wiki/API-Reference#qasyncgeneratorfunction)
tool for creating promised functions from generators.

And that's it.  Three simple functions, and you can have a form submitting
data to your Fieldbook.

If you go to the URL for your codelet you should see a nice form:

![codelet-form-tacit](../images/codelet-form-tacit.png)

Extensions
----------

There are a lot of additions that could be added to this tiny codelet:

* We don't do any validation of country; we could
  force the form to validate the Country name out of a list.

* We could also consult external services using `requestify`.

* If we had multiple linked sheets, we could use the data from the POST
  to set up records in multiple sheets and link them together.

All the code
------------

Here is all the code in one place suitable for pasting into your own book.

```javascript
var Q = require('q');
exports.endpoint = function (req, res) {
  if (req.method === 'GET') {
    return renderForm(req, res);
  } else if (req.method === 'POST') {
    return postForm(req, res);
  } else {
    throw new Error('Bad method: ' + req.method);
  }
}

var renderForm = function (req, res) {
  res.type('text/html')
  return `
    <head><link rel="stylesheet" href="//yegor256.github.io/tacit/tacit.min.css"/></head>
    <body>
      <form method="post">
        Name: <input name="name" type="text">
        Country: <input name="country" type="text">
        <input type="submit">
      </form>
    </body>
  `;
}

var postForm = Q.async(function * (req, res) {
  yield client.create('people', req.params);
  return 'Thanks!'
})
```
