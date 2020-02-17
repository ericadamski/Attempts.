---
title: Simple Password Management
date: "2020-02-17T16:51:50.250Z"
description: "How to use paassword.now.sh to manage passwords in a React NextJS app."
---

Here I will show you how you can add simple password management to your React application in minutes using [paassword.now.sh](https://paassword.now.sh). In this article I will be using React and the [NextJS](https://nextjs.org) framework! I recorded a live stream doing this exact same thing for a personal project of my own, you can see it [here](https://www.youtube.com/watch?v=b_DcxjKUKOw)

<iframe width="560" height="315" src="https://www.youtube.com/embed/b_DcxjKUKOw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

First we are going to create our sign up / log in page by creating a file in the `pages` directory of our project, something like: `pages/sign-up.js`. Using NextJS this will now allow you to navigate to the route `/sign-up` in your browser. In that file we can add our form:

```JSX
// pages/sign-up.js

export default () => {
    const handleSubmit = async event => {
        event.preventDefault();
    }

    return (
        <>
            <h1>Log In</h1>
            <form onSubmit={handleSubmit}>
                <input
                    type="email"
                    name="email"
                    placeholder="Enter email"
                />
                <input
                    type="password"
                    name="password"
                    placeholder="Enter password"
                />
                <button type="submit">Let's go!</button>
            </form>
        </>
    )
}
```

Now we want to handle the submission of that form to create a new user or login a current user. For that we will need an api route, which I will call `/api/user/auth`. Here is the structure for that file:

```JavaScript
// pages/api/user/auth.js
// A service to connect to Mongo DB
import connectToDb from './services/connect-to-db';
// A Mongoose model for a user that contains an email and a password Id
import User from './models/User';

export default async (req, res) => {
    // Make sure we initiate our database connection
    connectToDb();

    // our plain text email and password from the form
    const { email, password } = req.body;

    // Send a 200 OK
    res.end();
}
```

To store our email and password we will need to create a `fetch` request to our api route.

```JSX{5-25}
// pages/sign-up.js
import fetch from 'fetch';

export default () => {
    const handleSubmit = async event => {
        event.preventDefault();

        const {
            email: emailElement,
            password: passwordElement
        } = event.target.elements;

        const email = emailElement.value;
        const password = passwordElement.value;

        const response = await fetch('/api/user/auth', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password })
        })

        if (response.ok) {
            // successfully created a new user
            // OR logged in!
        }
    }

    return (
        <>
            <h1>Log In</h1>
            <form onSubmit={handleSubmit}>
                <input
                    type="email"
                    name="email"
                    placeholder="Enter email"
                />
                <input
                    type="password"
                    name="password"
                    placeholder="Enter password"
                />
                <button type="submit">Let's go!</button>
            </form>
        </>
    )
}
```

In that handler we will want to create a new user! First we need to store and encrypt our password in [paassword.now.sh](https://paassword.now.sh). Then we can store the `id` that paassword returns in our own database to use later to verify password attempts.

```JavaScript{10-31}
// pages/api/user/auth.js
import fetch from 'isomorphic-unfetch';
import connectToDb from './services/connect-to-db';
import User from './models/User';

export default async (req, res) => {
    connectToDb();
    const { email, password } = req.body;

    // Store the password in paassword.now.sh
    const paasswordResponse = await fetch(
        'https://paassword.now.sh/api/create',
        {
            method: 'POST',
            headers: { 'Content-Type': 'application-json' },
            body: JSON.stringify({ pwd: password })
        }
    );

    if (paasswordRresponse.ok) {
        // get the id from the response
        const { id } = await paasswordResponse.json();

        // store the id and the email so we can log in later
        const user = new User({
            email,
            passwordId: id
        });

        await user.save();
    }

    res.end();
}
```

[Paassword](https://paassword.now.sh) uses [Airtable](https://airtable.com/) to store encrypted strings that are only referenceable by the `id` that is returned. You can learn more about how it works [here](https://attempts.space/paassword/) and see the open source code [here](https://github.com/ericadamski/serverless-password). Storing a secure password is as simple as one request like this:

```JavaScript
await fetch(
    'https://paassword.now.sh/api/create',
    {
        method: 'POST',
        headers: { 'Content-Type': 'application-json' },
        body: JSON.stringify({ pwd: password })
    }
);
```

That request returns us an `id` we can then validate a password against. Once it is stored in our database, using MongoDB in the example above, we can then reference by email and compare passwords with our `passwordId`.

Now if we want to check if someone has logged in, we can:

1. find their user record by looking up their email
2. use their `passwordId` to request a comparison from paassword

```JavaScript{10-39}
// pages/api/user/auth.js
import fetch from 'isomorphic-unfetch';
import connectToDb from './services/connect-to-db';
import User from './models/User';

export default async (req, res) => {
    connectToDb();
    const { email, password } = req.body;

    // Attempt to find a user with that email
    let user = await User.findOne({ email });

    if (user) {
        // We have found a user that matches the input email,
        // now we have to validate that the password entered
        // matches what we originally saved
        const validateResponse = await fetch(
            `https://paassword.now.sh/api/get/${user.passwordId}`,
            {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ pwd: password })
            }
        );

        if (validateResponse.ok) {
            const { valid } = await validateResponse.json();

            if (valid) {
                // The passwords match! send a 200 OK
                return res.end();
            }
        }

        // The passwords don't match or there has been
        // a network failure trying to reach paasswords
        // send unauthorized.
        return res.status(401).end();
    }

    const paasswordResponse = await fetch(
        'https://paassword.now.sh/api/create',
        {
            method: 'POST',
            headers: { 'Content-Type': 'application-json' },
            body: JSON.stringify({ pwd: password })
        }
    );

    if (paasswordRresponse.ok) {
        const { id } = await paasswordResponse.json();

        user = new User({
            email,
            passwordId: id
        });

        await user.save();
    }

    res.end();
}
```

There it is!

A relatively simple way to integrate password authentication in your React app. This does not cover handling front end tokens like [JWTs](https://jwt.io/) or cookies but they can easily be added on now that verifing passwords is complete. Let me know if you want a more concrete example of this working or want to me write a follow up about JWT and cookies.

Thanks for reading!
