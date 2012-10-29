<!--{
  title: 'Your first Deployd API',
  tags: ['tutorial', 'hello world']
}-->

## Your first Deployd API

If you haven't already, [install Deployd](installing-deployd.md).

You can create a new project with the following commands.

    dpd create hello-world
    cd hello-world
    dpd -d
    
This should create your API, run Deployd, and open up the dashboard.

![Dashboard](/tutorials/first-api/images/new-dashboard.png)

Create a new collection by selecting it from the Resource **+** dropdown. In the "Create New Collection" prompt, enter `/things`.

Give your 'things' collection a property of `name` and press enter. Most screens in the dashboard have keyboard shortcuts.

Add some data to the collection by opening up the data editor. Click the "Data" menu item under "Things". With the data editor open and the new row selected you can just start typing a name. Type: "foo" then hit enter, "bat" enter, "baz" enter.

Now you should have a data editor that looks like this.

![Dashboard](/tutorials/first-api/images/data-editor.png)

Click the "API" menu item to see documentation for accessing your collection. If you open up `http://localhost:2403/things` in a browser, you should see the following JSON.

![Dashboard](/tutorials/first-api/images/json.png)

Using Chrome (or a browser with a console) open up: `http://localhost:2403` and pull up your console. Try the following:

    dpd.things.get(console.log);
    dpd.things.get({$limit: 1}, console.log);
    dpd.things.get({name: 'foo'}, console.log);

You should see something like the following:

![Dashboard](/tutorials/first-api/images/console.png)

### More Tutorials and Examples

 - [Your first App](your-first-app.md)

