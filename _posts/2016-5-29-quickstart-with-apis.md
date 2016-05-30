---
layout: post
title: Quickstart with APIs
author: Nathan Leung
comments: true
tags: [javascript]
---

APIs are a way for the frontend of your application to communicate with the backend (in other words, a way for your user to get the data of your application). Consider a simple blog with some posts. These posts are likely stored in a database on the backend, and you probably have to run queries like this to access them:

```js
// app.js
// query to find all posts, {} = no restrictions or search parameters, so everything is returned
Post.find({}).then((posts) => {
  // send our posts back in json format with status
  res.json({
    // so we can check if the query was successful
    status: "SUCCESS",
    posts: posts
  });
});
};
```

Once we find our posts, in our code above we send the posts back to the user in JSON format. It'll look something like this (let's say that the JSON is sent after a GET request to `/posts`):

```js
// http://localhost:8080/posts
{
  "status": "SUCCESS",
  "posts": [
    {
      "title": "My first post!",
      "text": "hey!"
    },
    {
      "title": "My second post!",
      "text": "blogs are so cool!"
    },
    ... // more posts
  ]
}
```

This doesn't look to pretty, unfortunately. While we could work around this by perhaps sending HTML back in our response like below, that code doesn't look too pretty either:

```js
// app.js
// set route to /posts, when user visits localhost:8080 this code will be run
app.get('/posts', (req, res) => {
  Post.find({}).then((posts) => {
    let postHTML = "";
    // iterate over each post
    // (we want to add each post to the html)
    posts.forEach((post) => {
      // for each post add its contents to the html
      postHTML += `
        <div class="post">
          // insert post.title and post.text property into string
          <h2>${post.title}</h2>
          <p>${post.text}</h2>
        </div>
      `;
    });
    return `
      <!DOCTYPE html>
      <html>
        <head>
          <title>my blog!</title>
          <!-- boilerplate html -->
          <meta charset="utf-8">
          <meta name="viewport" content="width=device-width,initial-scale=1">
          <!-- bootstrap so it doesn't look too ugly -->
          <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
        </head>
        <body>
          <h1>My Blog</h1>
          // insert post html that we generated above after iterating over every post
          ${postHTML}
          <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
        </body>
      </html>
    `;
  });
};
// run server on port 8080 (accessible at http://localhost:8080);
app.listen(8080);
```

While it'll look better on the client side, putting HTML into our backend Javascript files opens up a whole new can of worms. What if we wanted to change the site structure sometime in the future, for example? We'd have to go into those backend files and edit the layout, potentially messing up our queries and backend logic. To solve this problem, we should separate our concerns: each file should only focus on either the backend or frontend. Here's how we can do this.

## Separation of Concerns with an API

We'll separate our frontend and our backend files using an API.

Here's our main index.html file:

```html
// index.html, will be shown at http://localhost:8080/index.html (and http://localhost:8080/ too!)
<!DOCTYPE html>
<html>
  <head>
    <title>my blog!</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
  </head>
  <body>
    <h1>My Blog</h1>
    <!-- We'll insert our posts here -->
    <div id="posts"></div>
    <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
    <!-- See below, this will parse the data from the API and create an HTML list of our posts -->
    <script src="/scripts.js"></script>
  </body>
</html>
```

Here's the file we'll use to get the blog posts, on the server side. It'll simply get the data and send it to the client in raw format. (This is essentially our API, and how our frontend will get the post data).

```js
// app.js
app.get('/posts', () => {
  // query to find all posts, will be sent back after a GET to /posts
  Post.find({}).then((posts) => {
    // send our posts back in json format with status
    res.json({
      status: "SUCCESS",
      // shorthand for posts: posts
      posts
    });
  });
}
app.listen(8080);
```

And here's the file we'll use (on the frontend) to generate the HTML for the posts. We'll use jQuery and AJAX to hit our API located at `/posts`.

```js
// scripts.js (available at http://localhost:8080/scripts.js)
// We use AJAX to send a GET request to our API endpoint, located at /posts
$.get('/posts', (data) => {
  if (data.status === 'SUCCESS') {
    const posts = data.posts;
    let postHTML = '';
    posts.forEach((post) => {
      // for each post add its contents to the html
      postHTML += `
        <div class="post">
          // insert post.title and post.text property into string
          <h2>${post.title}</h2>
          <p>${post.text}</h2>
        </div>
      `;
    });
    // target the element with id 'posts' and fill it with the post html
    $('#posts').html(postHTML);
  }
});
```

We now have 3 files: `index.html` contains the site structure, `app.js` contains the backend logic and our API code (which is located at `/posts`, and `scripts.js` accesses this API and generates the HTML structure of the individual posts.

## Summary

Notice how the code is pretty much the same as what we had in the beginning, but it's just separated into different files. While it may not seem like much, now our code is a lot easier to maintain, and we can easily make modifications to each aspect of the blog. We can now easily change the blog layout (in `index.html`), easily change how posts are displayed (in `scripts.js`), and easily change which posts are sent to the client (in `app.js`, for example, if we were to filter the latest posts or posts with certain keywords in a search feature).