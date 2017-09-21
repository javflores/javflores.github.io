---
layout: post
title: React London, September 2017 meetup
---

The September meetup of React London was held on 20th September in the Facebook offices.
Here is a brief description of each topic:

## Lightweight GraphQL

Simple talk about the basics of GraphQL. Why do we care about it and starting from scratch implementing an app that talk GraphQl and React.
Create a HOC that can communicate with the server in GraphQL and them he went through Querys and Mutations.
Subscriptions are another type that allows the client being notified when an event occurs.

Another thing very interesting I didn't know was that if you have a React component composed of other child components, if we define a single graphql query for everything, if there is a change in the state that will trigger a query for all the data.
This is not very efficient. Relay and Apollo provide Fragments that we can define for every component, making it more efficient.
Parent will see what has changed and trigger the fragment that needs to be queried.

Finally the typical mutationId that I don't know what it is, it is actually called Global object identifier or GraphqlId, and it is a cache key.

Slides: [https://www.graphql-training.com/lightweight-graphql-react/#1](https://www.graphql-training.com/lightweight-graphql-react/#1)


## CSS in React, spoilt for options

React we know is component-based, that doesn't go well with CSS global nature. But CSS is still the best mechanism to style your page.
It is actually quite powerful and indeed Facebook suggested inline styles with React but we loose some of the best features of CSS with inline styles.

Solutions:

### CSS modules

[Documentation](https://github.com/css-modules/css-modules)

What we use at Findmypast. Can define a css to be used in a particular page only and import it in the react component, the JS and the css live together in the same folder in harmony.
They mentioned this package that I don't know if we use in FMP https://www.npmjs.com/package/classnames. I guess is similar to joinable by @rkotze.

### Styled components

[Documentation](https://github.com/styled-components/styled-components)

Very different, it a combo of JS and CSS syntax. Honestly it looks a bit weird, but we can give it a try.

### Glamorous

[Documentation](https://github.com/paypal/glamorous)

Write your styles in JS syntax, similar to inline styles but you get all the powerful features of CSS.
Looks interestinig. My issue is that, what happens if you don't like Glamorous, how do you go back to CSS? Would you have to rewrite everything again in CSS?

## ReasonML

[Documentation](https://github.com/facebook/reason)

These are all my notes of this talk:

**THIS IS A COPY OF ELM. WHY?! WHY?!**

It's ok, yes, it is a copy, but how many things do we copy or back and forth in software. A shiny hyped company copies the work of a PhD student that no one knows.
ReasonML is the shiny new language by Facebook, that's why it is getting the hype. It has been created by one of the original members that createdd React, that's why it is hot.
They are working hard to make it work really well with React and have Redux-style functionality built in ReasonML so that you don't need it anymor (Elm Architecture?!! *ehemehm* :D).

In the talk the speaker went through some of the features, basically functional programming inspired by OCaml, like Elm.

Maybe I take a look at it and according to the speaker everyone is going to write their React in ReasonML in one year time.
Personally I doubt it. In my mind I'd rather keep on practicing my Elm instead of learning this copy that is less polished and less developer-happiness origeted. :D
Sorry if you noticed some not good mood in there. At the end of the day is technology, and one shouldn't get too emotional with it.

This is the video of the whole talk:
[https://www.facebook.com/pg/RedBadger/videos/?ref=page_internal](https://www.facebook.com/pg/RedBadger/videos/?ref=page_internal)

