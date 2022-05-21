# rest2

> A simple REST application

## About

This project uses [Feathers](http://feathersjs.com). An open source web framework for building modern real-time applications.

## Getting Started

Getting up and running is as easy as 1, 2, 3.

1. Make sure you have [NodeJS](https://nodejs.org/) and [npm](https://www.npmjs.com/) installed.
2. Install your dependencies

    ```
    cd path/to/rest2
    npm install
    ```

3. Start your app

    ```
    npm start
    ```

## Testing

Simply run `npm test` and all your tests in the `test/` directory will be run.

## Scaffolding

Feathers has a powerful command line interface. Here are a few things it can do:

```
$ npm install -g @feathersjs/cli          # Install Feathers CLI

$ feathers generate service               # Generate a new Service
$ feathers generate hook                  # Generate a new Hook
$ feathers help                           # Show all commands
```

## Help

For more information on all the things you can do with Feathers visit [docs.feathersjs.com](http://docs.feathersjs.com).

## Begin Here

Install the dependencies and start the application:

    npm install
    PORT=8080 npm run dev

Preview the running application and verify that the signup page appears in the browser.

Stop the application with Ctrl-C.

### Create REST Service Backend

Install the feathers CLI:

    npm i -g @feathersjs/cli
    

Create a new service for the application to manage “signups”:

    feathers generate service

Answer the questions as follows:

    What kind of service is it? Mongoose
    What is the name of the service? signups
    Which path should the service be registered on? /signups
    What is the database connection string? MONGODBURI
    

`MONGODBURI` is an environment variable containing the MongoDB connection string. You will have to set this before running the application, e.g.

    export MONGODBURI='mongodb+srv://...'
    

The generated file, `src/mongoose.js`, is missing a connection setting. Add `useUnifiedTopology: true` to the list of mongodb connection settings.

In your editor, open up `src/models/signups.model.ts`.

There is currently a Mongoose schema definition for signups that looks something like this:

      const schema = new Schema({
        text: { type: String, required: true }
      }, {
        timestamps: true
      });

Edit it so that it looks like this:

      const schema = new Schema({
        firstName: { type: String, required: true },
        lastName: { type: String, required: true },
        email: { type: String, required: true },
        country: { type: String, required: true },
        province: { type: String, required: true },
        postalCode: { type: String, required: true }
      }, {
        timestamps: true
      });

### Create REST Client Frontend

In `public/index.html` add the following lines toward the end of the file after `bootstrap.min.js` but before `form-validation.js`:

    <script type="text/javascript" 
            src="//cdnjs.cloudflare.com/ajax/libs/core-js/2.1.4/core.min.js"></script>
    <script src="//unpkg.com/@feathersjs/[email protected]^3.0.0/dist/feathers.js"></script>

In `public/form-validation.js` add the following code at the beginning of the `load` event listener to set up the `signups` service in the client:

        // Set up FeathersJS app
        var app = feathers();
        
        // Set up REST client
        var restClient = feathers.rest();
    
        // Configure an AJAX library with that client 
        app.configure(restClient.fetch(window.fetch));
    
        // Connect to the `signups` service
        const signups = app.service('signups');

A little further down in `public/form-validation.js` there is code to validate the form fields and nothing else:

          form.addEventListener('submit', function (event) {
            if (form.checkValidity() === false) {
              event.preventDefault()
              event.stopPropagation()
            }
            form.classList.add('was-validated')
          }, false)

Modify it as follows to use the `signups` service to create a new signup when the form is valid:

          form.addEventListener('submit', function (event) {
            if (form.checkValidity()) {
              signups.create({
                firstName: $('#firstName').val(),
                lastName: $('#lastName').val(),
                email: $('#email').val(),
                country: $('#country').val(),
                province: $('#province').val(),
                postalCode: $('#postalCode').val()
              });
              form.classList.remove('was-validated');
              form.reset();
            } else {
              form.classList.add('was-validated');
            }
            event.preventDefault();
            event.stopPropagation();
          }, false);

Re-start the backend:

    PORT=8080 npm run dev

Preview the applicaition and load in its own browser tab.

Fill out the form and submit.

You can verify that the data got added to the MongoDB database by looking at the collection in the MongoDB Atlas console.

### Read Data from REST Service

Next, we will modify the client-side app so that it displays all the current signups in the table at the bottom of the page.

First, at the top of `public/form_validation.js` add a function at the very top of the file that adds a single row to the table:

    // Adds a signup row to the table
    const addSignup = signup => {
      $('#signups > tbody:last-child').append(
        `<tr>
          <td>${signup.firstName}</td>
          <td>${signup.lastName}</td>
          <td>${signup.email}</td>
          <td>${signup.country}</td>
          <td>${signup.province}</td>
          <td>${signup.postalCode}</td>
        </tr>`
      );
    };

Then, add a function that will fetch the all of the signups from the server and then add them one at a time to the table:

    // Shows the signups
    const showSignups = async signupService => {
      // Find the latest 25 signups. They will come with the newest first
      const signups = await signupService.find({
        query: {
          $sort: { createdAt: -1 },
          $limit: 25
        }
      });
      
      // We want to show the newest signup last
      signups.data.reverse().forEach(addSignup);
    };

Finally, call the `showSignups` function on page load. In the `load` event listener, add the following code just after the connection to the `signups` service has been established.

        // Show existing signups in the table
        showSignups(signups);

