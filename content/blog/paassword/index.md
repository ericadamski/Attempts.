---
title: PaaS-sword
date: "2020-02-08T05:04:43.646Z"
description: "I wrote code for the first time during the day since starting my new job, and I ended up creating a few fun things."
---

My new role at work doesn't involve coding. I went from coding 8+ hours a day, mashing at keys, to answering emails and writing documents - a different type of key mashing. But today I got the chance to write some code, and it was **SUBLIME**!

I am working on changing how hiring is done at my company, shifting the perspective away from skills and onto culture and values. (If you are interested in my stance you can get a brief idea from [these slides](https://technical-hiring.now.sh/)) Since I am still a programmer at heart I feel way more creative using [mdx-deck](https://github.com/jxnblk/mdx-deck) to create my slide decks. It is a hybrid [React](https://reactjs.org/) and [Markdown](https://en.wikipedia.org/wiki/Markdown) based presentation tool that allows me to manipulate, in finer detail, every aspect of the presentation. Does it take longer? **YES**. But it is significantly more enjoyable for me and it keeps me engaged. So here I am writing this slide deck for values based hiring training. I want each attendee to have easy access to the slides. I figured the internet was the easiest way to share them - since I'm programming anyway. **BUT** there is a huge problem. The presentation is for internal company use only - ⚠️**CONFIDENTIAL**⚠️! I was stuck. How can I put up my slides and keep them confidential? My answer: **Passwords**.

I thought I should password protect this presentation and that way everyone can have easy access to the presentation from the internet, at the same time being protected against wandering eyes. I spent 4 hours of my day solving this problem - _how can I password protect a mdx slide deck_? Since it uses React I figured the interface work would be simple, and truthfully there wasn't much to it. An input field, a label and a button to unlock the slides. The hard pard was figuring out how am was going to make sure the password was safe. I could hard code it into the presentation, but that doesn't seem secure enough. Which then means I have to keep it in some sort of environment variable. But I only have a front end I cannot use hidden secrets in environment files. I don't really want to have to write a complete backend for my slide deck! So I was stuck; _how do I store a password, **SECURELY**, and validate against it without keeping it in the source code_? 💥**BOOM**💥 then comes an idea!

I decide that encrypting and storing passwords doesn't need a lot of technology. It needs:

1. Some way of setting a password
2. Some place to store the encrypted version of the password
3. Some way of comparing an attempt to the encrypted version of the password

With all the tools that exist today I had to do very little work or setup to cross those 3 requirements off of my list. I decided I would write 2 [serverless](https://en.wikipedia.org/wiki/Serverless_computing) functions, one to handle creating, encrypting and storing a new password and one to compare passwords. I used the amazing products provided by [ZEIT](https://now.sh) to write and host my functions. And the _"database"_ for the encrypted passwords? I used [Airtable](https://airtable.com/). With these two technologies I was able to go from idea, to a completely working service in less than an hour!

> You can check out the code [here](https://github.com/ericadamski/serverless-password/tree/master/api), it's open source!

In case you are worried about sending your passwords to some random persons Airtable, I don't blame you. Honestly, this is all I am storing!

<img width="424" alt="encrypted passwords" src="https://user-images.githubusercontent.com/6516758/74079764-dbbd5180-4a09-11ea-92f6-d59b4de46064.png">

There is no information in that table other than the encrypted password.

After this fun little foray of creating what I am calling a `Password as a Service` 😂 tool, I got right to creating my password protected presentation! The code for this is also open source and you can check it out [here](https://github.com/ericadamski/protect-a-deck/blob/master/index.js). I found it worked so well that I had to create a package for other people to use, so now you can also password protect your mdx-deck presentations with `protect-a-deck` 😂 (I am on fire with names right now 🔥).

Let me explain how this all works!

First, you come to my wonderful, publicly accessible website where the presentation is. The code checks to see if you have validated yourself, if you have not it doesn't show you any of the slide content.

In React something like this:

```JavaScript
<div>
    {valid ? ( props.children ) : ( /* lock screen */ )}
</div>
```

The content of the slides isn't rendered and so it cannot be inspected with developer tools. **You just can't see it!**. Once you enter in your password I send off a request, to the handy new service I created, which checks against the password I set to see if they match. If they do, **YOU'RE IN**. Otherwise I send a very straight forward message kindly letting you know that you didn't make it.

<img width="362" alt="example failed password validation" src="https://user-images.githubusercontent.com/6516758/74079890-618dcc80-4a0b-11ea-87bb-a7302fa9ee6b.png">

It was so smooth I thought that I have to share this with other people! Not just the code, but also just a nice way people could create and easily validate their own secure passwords. So I popped a UI onto my two serverless functions to help create and compare passwords. I called it, because I am so good at naming 😂, [PaaS-sword](https://paassword.now.sh). You can head over there and start comparing passwords!

Before I let you go, let me give you an extremely quick rundown of how [PaaS-sword](https://paassword.now.sh) works.

1. You submit a plain text password (don't worry I use `https`, so it is somewhat safe in transit).
2. My first serverless function uses [bcrypt](https://www.npmjs.com/package/bcrypt) to encrypt the plain text password.
3. I store the encrypted password in the Airtable, **BOOM** stored.
4. I then return the Airtable reference to the row that the encrypted password exists in so we can compare against it later.

Now when I want to compare them,

1. I send a `POST` request with the Airtable reference from above (on the site I give you a nice URL for it) and some new plain text password to compare against.
2. I get the encrypted password from the Airtable and compare that with the plain text password you just sent.
3. If bcrypt says they match, **HOORAY**, if not, too bad.

My severs store nothing, the only thing that persists in Airtable is a fun string like:

> $2b$10\$2o2z7RbWDeuEw.3sH4jYBuRbsQXVV9keK.83jcw01uQGjMiTY3cUW

Which means nothing to nobody and can never be translated back into plain text.

All of this to say I had a blast being creative about solving my problems. It is amazing the tools that exist to solve even seemingly trivial problems. These tools are only hours old, they have a long way to go before they fulfill their full potential. If you use them, and you like them, let me know so I can continue to make them better!

This was written at 1AM, so please - be kind to the tired rambling version of myself 😂

Thank you for reading!
