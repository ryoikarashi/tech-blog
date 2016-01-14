---
layout: post
title: Making API without DB [Part 2]
date: 2016-01-09
---

# In this post...
We'll make routes to access to resource.

# Routes

In the previous post, we designed the routes that we're going to create.

Here is the routes again.

```json
{
  "GET /users": {
    "desc": "returns all users",
    "response": "200 application/json",
    "data": [{}, {}, {}]
  },

  "GET /users/:id": {
    "desc": "returns one user respresented by its id",
    "response": "200 application/json",
    "data": {}
  },

  "POST /users": {
    "desc": "create and returns a new user uisng the posted object as the user",
    "response": "201 application/json",
    "data": {}
  },

  "PUT /users/:id": {
    "desc": "updates and returns the matching user with the posted update object",
    "response": "200 application/json",
    "data": {}
  },

  "DELETE /users/:id": {
    "desc": "deletes and returns the matching user",
    "response": "200 application/json",
    "data": {}
  }
}
```

### Adding Middleware and global variables
```js
'use strict';

// require 3rd party libs
const express = require('express');
const bodyParser = require('body-parser');
const _ = require('lodash');

// create express app
const app = express();

// port
const port = 3000;

// global variables
let users = [];
let id = 0;

// Another middleware to add unique id to each user
let updateId = (req, res, next) => {
  if(!req.body.id) {
    id++;
    req.body.id = id + '';
  }
  next();
};

// set middleware
app.use(express.static('client'));
app.use(bodyParser.urlencoded({extended: true}));
app.use(bodyParser.json());
// error-handling middleware
app.use((err, req, res, next) => {
  if(err) {
    res.status(500).send(err);
  }
});

app.param('id', (req, res, next, id) => {
  let user = _.find(users, {id: id});
  if(user) {
    req.user = user;
    next();
  } else {
    res.send();
  }
});
```

### GET /users

```js
...

app.get('/users', (req, res) => {
  res.json(users);
});

...
```

### GET /users/:id

```js
...

app.get('/users/:id', (req, res) => {
  let user = req.user;
  res.json(user || {});
});
...
```

### POST /users

```js
...

app.post('/users', (req, res) => {
  let user = req.body;
  users.push(user);
  res.json(user);
});

...
```

### PUT /users/:id

```js
...

app.put('/users/:id', updateId, (req, res) => {
  let update = req.body;

  if(update.id) {
    delete update.id
  }

  let user = _.findIndex(users, {id: req.params.id});
  if(!user) {
    res.send();
  } else {
    let updatedUser = _.assign(users[user], update);
    res.json(updatedUser);
  }
});

...
```

### DELETE /users/:id

```js
...

app.delete('/users/:id', (req, res) => {
  let deletedUserIndex = _.findIndex(users, {id: req.params.id});
  let deletedUser = users[deletedUserIndex];
  users.splice(deletedUserIndex, 1);
  res.json(deletedUser);
});

...
```

Then listen on port 3000.

```js
...

app.listen(port, () => {
  console.log('listening on port ', port);
});
```

# Execute HTTP methods
Finally we can execute http methods (*GET*/*POST*/*PUT*/*DELETE*) on our terminal app.
To do so, first of all, run the server.

```js
node server/server.js

listening on http://localhost:3000
```

Then install [httpie](https://github.com/jkbrzt/httpie) to execute http methods.
>HTTPie (pronounced aitch-tee-tee-pie) is a command line HTTP client. Its goal is to make CLI interaction with web services as human-friendly as possible. It provides a simple http command that allows for sending arbitrary HTTP requests using a simple and natural syntax, and displays colorized output. HTTPie can be used for testing, debugging, and generally interacting with HTTP servers.

source: [HTTPie: a CLI, cURL-like tool for humans](https://github.com/jkbrzt/httpie#httpie-a-cli-curl-like-tool-for-humans)

Now we're ready to go!

Create two users.

### POST /users
```bash
http POST http://localhost:3000/users name=ryo

...
HTTP/1.1 200 OK
...

[
    {
        "id": "1",
        "name": "ryo"
    }
]
```

```bash
http POST http://localhost:3000/users name=sample

...
HTTP/1.1 200 OK
...

[
    {
        "id": "2",
        "name": "sample"
    }
]
```

Get two users that you created before.

### GET /users
```bash
http GET http://localhost:3000/users

...
HTTP/1.1 200 OK
...

[
    {
        "id": "1",
        "name": "ryo"
    },
    {
        "id": "2",
        "name": "sample"
    }
]
```

Get a user with a certain ID

### GET /users/:id
```bash
http GET http://localhost:3000/users/1

...
HTTP/1.1 200 OK
...

{
    "id": "1",
    "name": "ryo"
}
```

Update user's info with a certain ID

### PUT /users/:id
```bash
http PUT http://localhost:3000/users/1 name=ryo-modified

...
HTTP/1.1 200 OK
...

{
    "id": "1",
    "name": "ryo-modified"
}
```

Delete an user with a certain ID

### DELETE /users/:id
```bash
http DELETE http://localhost:3000/users/1

...
HTTP/1.1 200 OK
...

[
    {
        "id": "1",
        "name": "ryo-modified"
    }
]
```

Works perfect!
Now you know it's really easy to design and create API with express!

# Next...
In the next post, we'll a little bit more complicated app with DB.