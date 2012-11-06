<!--{
  title: 'Event API',
  tags: ['reference', 'collection']
}-->

## Event API

### this <!--ref-->

The current object is represented as `this`. You can always read its properties. Modifying its properties in an `On Get` request will change the result that the client recieves, while modifying its properties in an `On Post`, `On Put`, or `On Validate` will change the value in the database.

    // Example: On Validate
    // If a property is too long, truncate it
    if (this.message.length > 140) {
      this.message = this.message.substring(0, 137) + '...';
    }

*Note*: In some cases, the meaning of `this` will change to something less useful inside of a function. If you are using functional programming (such as `Array.forEach()`), you may need to bind another variable to `this`:

    // Won't work - sum remains at 0
    this.sum = 0;
    this.targets.forEach(function(t) {
      this.sum += t.points;
    });
    
<!--seperate-->

    //Works as expected
    var self = this;

    this.sum = 0;
    this.targets.forEach(function(t) {
      self.sum += t.points;
    });

### me <!-- ref -->

The currently logged in User from a User Collection. `undefined` if no user is logged in.

    // Example: On Post
    // Save the creator's information
    if (me) {
        this.creatorId = me.id;
        this.creatorName = me.name;
    }
### query <!-- ref -->

The query string object. On a specific query (such as `/posts/a59551a90be9abd8`), this includes an `id` property.

    // Example: On Get
    // Don't show the body of a post in a general query
    if (!query.id) {
      hide(this.body);
    }

### cancel() <!-- ref -->

    cancel(message, [statusCode])

Stops the current request with the provided error message and HTTP status code. Status code defaults to `400`. Commonly used for security and authorization. 

It is strongly recommended that you `cancel()` any events that are not accessible to your front-end, because your API is open to anyone.

    // Example: On Post
    // Don't allow non-admins to create items
    if (!me.admin) {
      cancel("You are not authorized to do that", 401);
    }

*Note: In a GET event where multiple values are queried (such as on `/posts`), the `cancel()` function will remove the current item from the results without an error message.*

### error() <!-- ref -->

    error(key, message)

Adds an error message to an `errors` object in the response. Cancels the request, but continues running the event so it can to collect multiple errors to display to the user. Commonly used for validation.

    // Example: On Validate
    // Don't allow certain words
    // Returns response {"errors": {"name": "Contains forbidden words"}}
    if (!this.name.match(/(foo|bar)/)) {
      error('name', "Contains forbidden words");
    }

### hide() <!-- ref -->

    hide(property)

Hides a property from the response.

    // Example: On Get
    // Don't show private information
    if (!me || me.id !== this.creatorId) {
      hide('secret');
    }

### protect() <!-- ref -->

    protect(property)

Prevents a property from being updated. It is strongly recommended you `protect()` any properties that should not be modified after an object is created. 

    // Example: On Put
    // Protect a property
    protect('createdDate');

<!-- seperate -->

    // Example: On Put
    // Only the creator can change the title
    if (!(me && me.id === this.creatorId)) {
      protect('title');
    }
    

### changed() <!-- ref -->

    changed(property)

Returns whether a property has been updated.

    // Example: On Put
    // Validate the title when it changes
    if(changed('title') && this.title.length < 5) {
      error('title', 'must be over 5 characters');
    }
    
### previous <!-- ref -->

An `Object` containing the previous values of the item to be updated.

    // Example: On Put
    if(this.votes < previous.votes) {
      emit('votes have decreased');
    }

### emit() <!-- ref -->

    emit([userCollection, query], message, [data])

Emits a realtime message to the client
You can use `userCollection` and `query` parameters to limit the message broadcast to specific users.

    // Example: On Put
    // Alert the owner that their post has been modified
    if (me.id !== this.creatorId) {
      emit(dpd.users, {id: this.creatorId}, 'postModified', this); 
    } 

<!--seperate-->

    // Example: On Post
    // Alert clients that a new post has been created
    emit('postCreated', this);

In the front end:

    // Listen for new posts
    dpd.on('postCreated', function(post) {
        //do something...
    });

See [Notifying Clients of Changes with Sockets](../notifying-clients.md) for an overview on realtime functionality.

### dpd <!-- ref -->

The entire [dpd.js](/docs/reference/dpdjs.html) library, except for `dpd.on()`, is available from events. It will also properly bind `this` in callbacks.

    // Example: On Get
    // If specific query, get comments
    dpd.comments.get({postId: this.id}, function(results) {
      this.comments = results;
    });

<!--seperate-->

    // Example: On Delete
    // Log item elsewhere
    dpd.archived.post(this);

Dpd.js will prevent recursive queries. This works by returning `null` from a `dpd` function call that has already been called several times further up in the stack. You can change this with the [$limitRecursion](./querying-collections.md#s-$limitRecursion) property

    // Example: On Get /recursive
    // Call self
    dpd.recursive.get({$limitRecursion: 1}, function(results) {
        if (results) this.recursive = results;
    });

<!--seperate-->

    // GET /recursive
    {
        "id": "a59551a90be9abd8",
        "recursive": [
            {
                "id": "a59551a90be9abd8"    
            }
        ]
    }


### internal <!-- ref -->

Equal to true if this request has been sent by another script.

    // Example: On GET /posts
    // Posts with a parent are invisible, but are counted by their parent
    if (this.parentId && !internal) cancel();

    dpd.posts.get({parentId: this.id}, function(posts) {
        this.childPosts = posts.length;
    });

### isRoot <!-- ref -->

Equal to true if this request has been authenticated as root (has the `dpd-ssh-key` header with the appropriate key; such as from the dashboard)

    // Example: On PUT /users
    // Protect reputation property - should only be calculated by a custom script.

    if (!isRoot) protect('reputation');


### console.log() <!-- ref -->

    console.log([arguments]...)

Logs the values provided to the command line. Useful for debugging.