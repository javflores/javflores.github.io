---
layout: post
title: Workshop Introduction to Elm
---

Notes of my workshop.

## Presentation of Elm

I ended up looking into Elm because I listened a podcast by the Redux creator, and he got the inspiration from Elm.
Then I looked into Elm and I liked type safety and functional programming approach in the browser.
But don't waste time on this explanation, let's do that some work!

Who has written any code in Elm? Who knows what Elm is?
How many people heard about Babel? Who has used Babel?
Babel lets you start next generation Javascript today. Let's start by writting a function called pluralize in [Babels repl](https://babeljs.io/repl/):

{% highlight js %}
function pluralize(singular, plural, quantity){
  if(quantity === 1){
    return singular;
  }
  else{
    return plural;
  }
}
{% endhighlight %}

Let's do a console log:

{% highlight js %}
console.log(pluralize("shelf", "shelves", 3))
{% endhighlight %}

So it is going to return the singular or the plural of a given word.

Let's redefine the function in the new Javascript language using the arrow function:

{% highlight js %}
var pluralize = (singular, plural, quantity) => {
  if(quantity === 1){
    return singular;
  }
  else{
    return plural;
  }
}
{% endhighlight %}

Same behaviour, you may notice that in the right we have something different. All the browsers will support the right side.

Elm does a similar thing but instead, what you put on the left is a very different language. If we navigate to [http://elm-lang.org/try](http://elm-lang.org/try) and select Hello World, now let's implement pluralize function:

{% highlight elm %}
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)

pluralize singular plural quantity =
  if quantity == 1 then
    singular
  else
    plural
    
main =
  text (pluralize "shelf" "shelves" 2)
{% endhighlight %}

A couple of things are different.
- function syntax: name of function, follow by parameters, no parentheses.
- if syntax is different
- you have == instead of === 
- we don't have return because it is implicit as it is in Elixir

Why do we bother writing Elm if we can write JS. Let's see if we can do something that Javascript can't do for us.

We did a typo, we put singula.
{% highlight js %}
var pluralize = (singular, plural, quantity) => {
  if(quantity === 1){
    return singula;
  }
  else{
    return plural;
  }
}

console.log(pluralize("shelf", "shelves", 3))
{% endhighlight %}

No issue, but:
{% highlight js %}
console.log(pluralize("shelf", "shelves", 1))
{% endhighlight %}

Getting the common undefined issue. The trouble is that if we don't call that path, we'll have no idea, this is a bomb waiting to explode in the future.

(PARTICIPANTS) In Elm, doing the same, the compiler tells about the possible typo. Elm is looking to all the values and it tries to match.


Another error, let's compare with quantity "1"

{% highlight js %}
var pluralize = (singular, plural, quantity) => {
  if(quantity === "1"){
    return singular;
  }
  else{
    return plural;
  }
}

console.log(pluralize("shelf", "shelves", 1))
{% endhighlight %}

We don't get an exception, but we get a wrong behaviour. It will be really hard to find this in production.
(PARTICIPANTS) Same in Elm... We get a type mismatch.

## Rendering Html

The main there is capable of not only rendering text but Html.
So let's do something a bit more fancy:

{% highlight elm %}
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)

pluralize singular plural quantity =
  if quantity == 1 then
    singular
  else
    plural
    
main =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , text (pluralize "shelf" "shelves" 2)
    ]
{% endhighlight %}

(Explain, this). The technology uses the same thing as React, the Virtual DOM, instead of rendering the whole thing, it renders the things that have changed. You don't mutate the DOM like in JQuery. It is different to React, it doesn't use JSX, it uses functions!
div, h1 and text are functions. They have two parameters, a list of attributes, and a list of child components. Each child works the same way, actually html is a linked list.

{% highlight elm %}
  div [class "content", id = "main-body"][]
{% endhighlight %}

## Interactivity

Let's add a button so we can increase the quantity interactively.
To do that, we need the main function a bit differently:

{% highlight elm %}
import Html.App as Html
{% endhighlight %}

Main needs a number of parameters:

- The init function:

{% highlight elm %}
init = 
  ({ quantity = 5 }, Cmd.none)
{% endhighlight %}

- An update function which takes a model and action:

{% highlight elm %}
update msg model =
  (model, Cmd.none)
{% endhighlight %}

- The view:

{% highlight elm %}
view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , text (pluralize "shelf" "shelves" 2)
    ]
{% endhighlight %}

- Finally a subscriptions function:

{% highlight elm %}
subscriptions model =
  Sub.none
{% endhighlight %}

Now we can define the main as:

{% highlight elm %}
main = Html.program
    { init = init,
    view = view,
    update = update,
    subscriptions = subscriptions
    }
{% endhighlight %}

Your view function is called everytime you rerender the page and tells how it should be represented. Update translates user actions (messages) into a new model and new message. The init represents the initial application state. 
In this case the initial data as a "Record", similar to objects but they don't have behaviour or inheritance, it is just data.
(Don't worry for now on the subscriptions function).

Now in the view we can replace the harcoded 5 with model.quantity:

{% highlight elm %}
view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , text (pluralize "shelf" "shelves" model.quantity)
    ]
{% endhighlight %}

EXERCISE: Can you show the quantity in the view?

Tip: toString converts a number into a string. 
Tip: Concatenating strings: " " ++ " "

POSSIBLE SOLUTIONS: 

{% highlight elm %}
view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , text ((toString model.quantity) ++ " ")
    , text (pluralize "shelf" "shelves" model.quantity)
    ]

view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , text (
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity))
    ]

{% endhighlight %}

Let's add the button to increment now:

{% highlight elm %}
view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , div [] 
      [ button [ ] [text "Increase" ]
      ]
    , text (
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity))
    ]

{% endhighlight %}

It doesn't do anything, we need the button to send a message (action) when it is clicked.
We create a Msg type:

{% highlight elm %}
type Msg = Increase
{% endhighlight %}

And we change the button:

{% highlight elm %}
[ button [ onClick Increase ] [text "Increase" ]
{% endhighlight %}

Now the button will be sending an Increase message. Let's make the update function handle that:

{% highlight elm %}
update msg model =
  if msg == Increase then
    ({model | quantity = model.quantity + 1 }, Cmd.none)
    
  else 
    (model, Cmd.none)
{% endhighlight %}

We are updating the model record with an increased quantity. Actually we are creating a new model, but with a different quantity.

(EXERCISE) It is a bit lame, now I can't go back, can you add a Decrease button?