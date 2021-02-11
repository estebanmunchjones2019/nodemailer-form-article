# React contact form with Nodemailer and cloud functions

Do you need to implement a contact form in a React app but don't want create a back-end app? Let's learn how to use an email service called [Nodemailer](https://nodemailer.com/about/), and [Cloud functions on Firebase](https://firebase.google.com/docs/functions), to make this task super easy! As a bonus you'll get a full working form with **validation** and with the style of **[Material UI](https://material-ui.com/)!



**Note: if you wanna deploy the cloud function to the cloud, you'll need a credit card, altough you're not gonna be charged unless you call the cloud function more than 2 million times**



Table of contents:

[Creating the contact form]



## Creating the contact form:

In this section we're gonna create a contact form with React and Material UI, and we're gonna add validation, a spinner to show that something is happending after clicking the submit button, and a snackbar to inform the user what was the result of the request.

So, let's get started and start building it:

1) **Create React app:**

````bash
npx create-react-app nodemailer-form
cd nodemailer-form
````

2) **Add [Material UI](https://material-ui.com/):**

````bash
npm install @material-ui/core
npm install @material-ui/lab
````

Please note that [react](https://www.npmjs.com/package/react) >= 16.8.0 and [react-dom](https://www.npmjs.com/package/react-dom) >= 16.8.0 are peer dependencies.

Material UI provides us with already styled React component we can use straight away, and they implement [Material](https://material.io/design/introduction), a design system created by Google to help teams build high-quality digital experiences for Android, iOS, Flutter, and the web.

In other words, using Material UI is gonna gives an App-like look to our form ;)

3) **Add Roboto Font:**

Material-UI was designed with the [Roboto](https://fonts.google.com/specimen/Roboto) font in mind.

One way of adding it is via a CDN: 

````html
<!-- src/index.html -->

<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" />
````

The above `<link>` tag must be added inside the `<head>` `</head>` parts.

Other way to install the font is via npm. You can learn how to do it [here](https://material-ui.com/components/typography/#general).

4) **Create the form**:

````javascript
//import useState hook
import { useState } from 'react';

//import styles
import './App.css';

//import Material UI components and functions
import TextField from '@material-ui/core/TextField';
import Grid from '@material-ui/core/Grid';
import Button from '@material-ui/core/Button';
import Box from '@material-ui/core/Box';
import CircularProgress from '@material-ui/core/CircularProgress';
import Snackbar from '@material-ui/core/Snackbar';
import { Alert } from '@material-ui/lab'; 
import { makeStyles } from '@material-ui/core/styles';

//add CSS classes for Material UI components calling a function that returns a another function
const useStyles = makeStyles((theme) => ({
  //the CSS class honeypot is later on added to the honeypot field, which is not displayed to users.
  honeypot: {
    display: 'none'
  }
}));  

function App() {
  //assign the constant classes to an object for Material IU components by calling a function
  const classes = useStyles();

  //define error state and the function to change it. The value is false by default
  const [error, setError] = useState(false); 

  //define openSnackbar state and the function to change it. The value is false by default
  const [openSnackbar, setOpenSnackbar] = useState(false);

  //define isLoading state and the function to change it. The value is false by default
  const [isLoading, setIsLoading] = useState(false); 

  //define formIsValid state and the function to change it. The value is false by default
  const [formIsValid, setFormIsValid] = useState(false); 

  //define contacForm state and the function to change it
  //the whole form is represented by an object, with nested objects inside that represent the input fields  
  const [contactForm, setContactForm] = useState({ 
    name: {
      value: '',
      elementConfig: {
        required: true,
        id: "standard-basic", 
        label: "Your Name",
        
      },
      validation: {
        required: true,
        errorMessage: 'Please, enter your name'
      },
      valid: false,
      blur: false
    },
  
    email: {
      value: '',
      elementConfig: {
        required: true,
        id: "standard-basic",
        label: "Your Email",
      },
      validation: {
        required: true,
        isEmail: true,
        errorMessage: 'Please, enter your email'
      },
      valid: false,
      blur: false
    },
  
    message: {
      value: '',
      elementConfig: {
        required: true,
        id: "standard-multiline-static",
        label: "Your Message",
        multiline: true,
        rows: 4,
      },
      validation: {
        required: true,
        errorMessage: 'Please, enter your message'
      },
      valid: false,
      blur: false
    },  
  
    //this honeypot field isn't rendered, so users don't see it, but fools bots that fill it automatically
    honeypot: { 
      value: '',
      elementConfig: {
        className: classes.honeypot,
        label: 'If you are a human, do not type anything here. I am here to fool bots', 
      },
      //This validation property is added just to avoid and error when running checkValidity();
      validation: {
      },
      //it's valid by default so it doesn't prevent human users to click the submit button
      valid: true,
      blur: false
    } 
  }); 

//convert the contactForm object into an array
const formElementsArray = []; 
for (let key in contactForm) {
    formElementsArray.push({
        id: key,
      ...contactForm[key]
    })
}

//map the array to return an array of JSX elements    
const formElements = formElementsArray.map(element => {
  return (
    <Box key={element.id}>
      <TextField
      {...element.elementConfig}
      error={!element.valid && element.blur} 
      helperText={(!element.valid && element.blur) ? element.validation.errorMessage : null} 
      onChange={(event) => inputChangedHandler(event, element.id)}  
      onBlur={(event) => inputChangedHandler(event, element.id)}  
      value={element.value}  
      ></TextField>  
    </Box>  
  );
}); 

//this function runs when an input changes or is blurred
const inputChangedHandler = (event, inputIdentifier)  => {

  //create a new object representing the input that was changed or blurred
  const updatedFormElement = {
    ...contactForm[inputIdentifier],
    value: event.target.value,
    valid: checkValidity(event.target.value, contactForm[inputIdentifier].validation),
    blur: event.type == 'blur' ? true : false,
    touched: true
  } 
  
  //create a new object represeting the whole form object
  const updatedContactForm = {
    ...contactForm,
    [inputIdentifier]:  updatedFormElement
  }
  
  //the whole form is valid until it's not
  let formIsValid = true;
  for (let inputElementIdentifier in updatedContactForm) {
      formIsValid = updatedContactForm[inputElementIdentifier].valid && formIsValid;
  }

  //update the contactForm state
  setContactForm(updatedContactForm);

  //update formIsValid
  setFormIsValid(formIsValid)

} 

//this function is called from inside inputChangedHandler(), and checks the validity of an input field
const checkValidity = (value, rules) => {

  //it's always true until there's one false in the way
  let isValid = true; 
  if (rules.required) {
    //value.trim() gets rid of white spaces
    isValid = value.trim() !== '' && isValid; 
  }

  if (rules.isEmail) {
    //the pattern is a Regular Expression that matches the shape of an email
    const pattern = /[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:[a-z0-9](?:[a-z0-9-]*[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]*[a-z0-9])?/;
    
    //pattern.test() returns true or false
    isValid = pattern.test(value) && isValid;
  }

  return isValid;

}

//this function is called when the users closes the snackbar after getting an error (when the cloud functions fails)
const closeSnackbar = () => {
  setOpenSnackbar(false);
}

//this function is called when clicking the "Send" button
const submitForm = () => {

  //if the bot filled the honeypot field, don't keep running code(e.g call a cloud function)
  if (contactForm.honeypot.value != '') {
    //the below return is called an "early return"
    return
  }

  //set isLoading to true, so the spinner is rendered
  setIsLoading(true);

  //this fake cloud function consoles log the data from the form that is passed to it, and has a 50% chance of resolving.
  const fakeCloudFunction = (data) => {
    console.log(data);
    return new Promise((resolve, reject) => {
      const error = Math.random() > 0.5 ? true : false;
      setTimeout(() => {
        if (!error) {
          resolve();
        } else {
          reject();
        }
      },1000)
    });
  }

   //call the fake cloud function. Later on this function will be replaced by the real cloud function.
   fakeCloudFunction({
    name: contactForm.name.value,
    email: contactForm.email.value,
    message: contactForm.message.value
  }).
  //this code below runs when the message was successfully sent from inside the cloud function
  then(() => {
    //create a new contactForm object that looks like the original contactForm state
    let originalContactForm = {};
    for(let key in contactForm){
      originalContactForm[key] = {
        ...contactForm[key],
        value: '',
        valid: key == 'honeypot'? true : false,
        blur: false
      }
    }

    //reset contactForm state to its original values
    setContactForm(originalContactForm);

    //reset the whole form validity to false
    setFormIsValid(false);

    //set error to false.
    setError(false);

    //set isLoading to false, so the spinner is not rendered anymore
    setIsLoading(false);

    //set openSnackbar to true, so the snackbar is rendered, with content that depends on the error state
    setOpenSnackbar(true);
  }).
  //this code below runs when the message was NOT successfully sent from inside of the cloud function
  catch(() => {
    //set the error state to true
    setError(true);

    //set isLoading to false, so the spinner is not rendered anymore
    setIsLoading(false);

    //set openSnackbar to true, so the snackbar is rendered, with content that depends on the error state
    setOpenSnackbar(true);
  })

} 
  

//return what's rendered to the virtual DOM
  return (
    <Box mt="3rem">
      <Grid container justify="center">  
        <Box>
          <Grid container alignItems="center" direction="column">
            <h2>Send us a message</h2>
            <p>We'll get back to you soon.</p>
            <form> 
              {formElements}
              <Grid container justify="center">
                <Box mt="2rem">
                  {isLoading ? <CircularProgress /> : (
                  <Button
                  onClick={submitForm}
                  disabled={!formIsValid} 
                  variant="contained"
                  color="primary"
                  >Send</Button>)} 
                </Box>            
              </Grid>
            </form>
          </Grid>
        </Box>
    </Grid>
    {error ? <Snackbar 
      open={openSnackbar} 
      onClose={closeSnackbar}
      anchorOrigin={{ vertical: 'top', horizontal: 'center' }}
      >
        <Alert onClose={closeSnackbar} severity="error">
          Oops! Something went wrong, try again later.
        </Alert>
      </Snackbar> : 
      <Snackbar 
      open={openSnackbar} 
      autoHideDuration={2000} 
      onClose={closeSnackbar}
      anchorOrigin={{ vertical: 'top', horizontal: 'center' }}
      >
      <Alert severity="success">
        Message sent!
      </Alert>
    </Snackbar>}
  </Box>
   
  );
}
export default App;


````

The above code has a dummy implementation of an cloud function, using a function with a `Promise` and a `setTimeout()`:

````javascript
//fakeCloudFunction

const fakeCloudFunction = (data) => {
    console.log(data);
    return new Promise((resolve, reject) => {
      const error = Math.random() > 0.5 ? true : false;
      setTimeout(() => {
        if (!error) {
          resolve();
        } else {
          reject();
        }
      },1000)
    });
  }
````

In the next section we're gonna add the real cloud function that actually sends and email with the data we provide in the form.

If you feel overwelmed by the length and complexity of the `App.js` React component containing the form, take a look a this Academind course: [React: The Complete Guide](https://pro.academind.com/p/react-the-complete-guide-incl-hooks-react-router-redux). The form code was copied from there, and it's explained step by step.

5) **Test the form:**

In order to test it, run this command:

````bash
npm start
````

and then open a tab in the browser and go to `localhost:3000` to see the form. Try clicking the fields and leave them empty, to see if validation works. Also try to add an invalid email and see what happens.

The form should let you submit it unless all the fields are not empty and the 

6) **Test the honeypot:**

As we don't want our form to be filled and submitted by bots, there's a trap we've set up in the form, which is a field that is not displayed to users, but that bots see, and fill it automatically. 

````javascript
//the honeypot field

 honeypot: { 
      value: '',
      elementConfig: {
        className: classes.honeypot,
        label: 'If you are a human, do not type anything here. I am here to fool bots', 
      },
      validation: {
      },
      valid: true,
      blur: false
 } 
````

````javascript
//don't call the cloud function if the field was filled

const submitForm = () => {
  if (contactForm.honeypot.value != '') {
    return
  }
  (...)
}
````

Then, we know that if the honeypot field was filled, that was a bot, and we handle the submission of the form in a different way, by doing nothing.

So, in order to test the honeypot, just change the `contactForm` state by swaping `contactForm.honeypot.value` from `''` to a number like `1` or a string `hey, I am bot who filled this form!`.

After doing the above change, you should still be able to click the `Send` button, but `fakeCloudFunction()` won't be called. Youd can verify that by looking at the `Console` on the dev tools on the browser. You shouldn't see the data from the form that is logged from inside `fakeCloudFunction()`.



Great! Now we have a form already working and manually tested! Let's now replace the `fakeCloudFunction()` with the real one in the following sections.



## Writing a cloud function for Firebase

#### What are they?

Cloud Functions for Firebase is a serverless framework that lets you automatically run backend code in response to events triggered by Firebase features and HTTPS requests. 

Your JavaScript or TypeScript code is stored in Google's cloud and runs in a managed environment. There's no need to manage and scale your own servers.

To know more about them, read the [docs](https://firebase.google.com/docs/functions).

#### Let's write one!

Follow these steps to write your first cloud function:

1) **Create a firebase project:**

In the [Firebase console](https://console.firebase.google.com/), click **Add project**, then select or enter a **Project name**.

If you wanna use an already existing project, [follow these instructions](https://firebase.google.com/docs/functions/get-started).

2) **Install the Firebase CLI:**

This installs the globally available firebase command. If the command fails, you may need to [change npm permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions). Type the following on the your terminal:

````bash
npm install -g firebase-tools
````

3) **Initialize your project:**



## Step 2: Register your app with Firebase

2) Go to general settings. Scroll down and select web app

3) 

````

````



## Step 3: Add Firebase SDKs and initialize Firebase

Using module bundlers

https://firebase.google.com/docs/web/setup#using-module-bundlers



1)  firebase install global tools



2) Install the Firebase CLI via `npm` by running the following command:

`````
npm install -g firebase-tools
`````

3) 

````
firebase login
````

````
firebase init
````

Choose:

? Which Firebase CLI features do you want to set up for this folder? Press Space to select features, then Enter to confirm your choic
es. (Press <space> to select, <a> to toggle all, <i> to invert selection)

````
Functions: Configure and deploy Cloud Functions
````

First, let's associate this project directory with a Firebase project.
You can create multiple project aliases by running firebase use --add, 
but for now we'll just set up a default project.

? Please select an option: (Use arrow keys)

````
Use an existing project
````



? Select a default Firebase project for this directory: 

````
nodemailer-form
````

A functions directory will be created in your project with a Node.js
package pre-configured. Functions can be deployed with firebase deploy.

? What language would you like to use to write Cloud Functions?

````
JavaScript 
````

 ? Do you want to use ESLint to catch probable bugs and enforce style? (y/N) 

````
N
````

The feedback is:

✔  Wrote functions/package.json
✔  Wrote functions/.eslintrc.js
✔  Wrote functions/index.js
✔  Wrote functions/.gitignore

? Do you want to install dependencies with npm now? (Y/n) 

```
y
```

✔  Firebase initialization complete!



Install nodemailer cors modules via npm:

````
cd functions
npm i nodemailer cors
````

why cors? Without it, calling the function from the react app throws this error:

````
Access to fetch at 'https://us-central1-nodemailer-form-8fdf0.cloudfunctions.net/sendEmail' from origin 'http://localhost:3000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
````

Because we're calling the function from the browser, from origin: localhost:3000. Wiht postman, we can always call the function, with or without cors.

On the developent phase, origin:true is OK, then add the url where your app is deployed to.

Now, we have to write a function:

````

````

what is nodemailer? https://nodemailer.com/about/



Test the function locally with a local emulator:

https://firebase.google.com/docs/functions/local-emulator

```
firebase emulators:start --only functions
```

 ✔  functions[sendEMail]: http function initialized (http://localhost:5001/nodemailer-form-8fdf0/us-central1/sendEmail).

Let's use postman:

Download postman in your computer:

https://www.postman.com/downloads/

See the steps

succes message

then turn internet off and see error message

Now, deploy the function to firebase

````
firebase deploy --only functions
````



````
const functions = require('firebase-functions');
const admin = require('firebase-admin');
const nodemailer = require('nodemailer');
const cors = require('cors')({origin: true});
admin.initializeApp();

let transporter = nodemailer.createTransport({
    host: "smtpout.secureserver.net",
    port: 465,
    secure: true, // true for 465, false for other ports
    auth: {
        user: 'hi@munchjones.com',
        pass: 'tebI9068.'
    }
});

exports.sendEmail = functions.https.onRequest((req, res) => {

    // Enable CORS using the `cors` express middleware.
    cors(req, res, () => {
      
        // getting dest email by query string
        const email = req.body.data.email;
        const name = req.body.data.name;
        const message = req.body.data.message;

        const mailOptions = {
            from: `hi@munchjones.com`,
            to: `hi@munchjones.com`,
            subject: 'New message from the app',
            text:  `New message from munchjones.com app. Sent by ${name} (${email}): ${message}`
           
        }
        // returning result
        return transporter.sendMail(mailOptions, (error, info) => {
            if(error){
                return res.status(500).send({
                    data:
                    {
                        "status": 500,
                        "message": error.toString()
                    }})
                }
            
            return res.status(200).send( {
                data:
                {
                    "status": 200,
                    "message": "sent"
            }});
        });
    });    
});


````



The following error when running:

````
firebase deploy --only functions
````

Error: Your project nodemailer-form-8fdf0 must be on the Blaze (pay-as-you-go) plan to complete this command. Required API cloudbuild.googleapis.com can't be enabled until the upgrade is complete. To upgrade, visit the following URL:



Now, to deploy the function, let's change the plan of the project. Cick on spark

Unles we hit call the function over 2M times a month, Google won't charge us at all!

Free up to 2M/month
Then $0.40/million

Super cheap!

Check the pricing here https://firebase.google.com/pricing?authuser=0

feedback:

✔  functions[sendEmail(us-central1)]: Successful create operation. 
Function URL (sendEmail): https://us-central1-nodemailer-form-8fdf0.cloudfunctions.net/sendEmail



Add a console.log(req) to the cloud function and pick it up from the log tab on firebase, for debugging purposes.



## Testing the cloud function locally

## Deploying the cloud function

## Testing the deployed cloud function

