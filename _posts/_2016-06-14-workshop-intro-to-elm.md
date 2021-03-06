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
TIP: To add another type of message:

{% highlight elm %}
type Msg = 
  Increase
  | Decrease
{% endhighlight %}

This is called Union type.

(SOLUTION)
You will have to update the Msg type, change the update function and add the button in the view:

{% highlight elm %}
type Msg = 
  Increase
  | Decrease

update msg model =
  if msg == Increase then
    ({model | quantity = model.quantity + 1 }, Cmd.none)
    
  else if msg == Decrease then
    ({model | quantity = model.quantity - 1 }, Cmd.none)
    
  else 
    (model, Cmd.none)

view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , div [] 
      [ button [ onClick Increase ] [text "Increase" ]
      , button [ onClick Decrease ] [text "Decrease" ]
      ]
    , text (
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity))
    ]
{% endhighlight %}

Great work! Any questions?

The number is going negative. Let's put the minimum to zero:

{% highlight elm %}
update msg model =
  if msg == Increase then
    ({model | quantity = model.quantity + 1 }, Cmd.none)
    
  else if msg == Decrease then
    ({model | quantity = Basics.max 0 (model.quantity - 1) }, Cmd.none)
    
  else 
    (model, Cmd.none)
{% endhighlight %}

Actually better, let's disable the button when it get's to zero:

{% highlight elm %}
view model =
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , div [] 
      [ button [ onClick Increase ] [text "Increase" ]
      , button [ onClick Decrease, disabled (model.quantity <= 0) ] [text "Decrease" ]
      ]
    , text (
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity))
    ]
{% endhighlight %}

We shouldn't have logic in our views right?
So let's move that check out of the button:

{% highlight elm %}
view model =
  let isDisabled =
    model.quantity <= 0
    
  in
  div [class "content" ]
    [ h1 [] [ text "Pluralizer" ]
    , div [] 
      [ button [ onClick Increase ] [text "Increase" ]
      , button [ onClick Decrease, disabled isDisabled ] [text "Decrease" ]
      ]
    , text (
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity))
    ]
{% endhighlight %}

Let binding works similar to Javascript, with one difference: it can't be reassigned (they act as constants). You can add as many properties as you like.

(EXERCISE) Can you move the text above into a caption defined in the let?
(SOLUTION)

{% highlight elm %}
view model =

  let 
    isDisabled =
      model.quantity <= 0
      
    caption = 
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity)   
    
  in
    div [class "content" ]
      [ h1 [] [ text "Pluralizer" ]
      , div [] 
        [ button [ onClick Increase ] [text "Increase" ]
        , button [ onClick Decrease, disabled isDisabled ] [text "Decrease" ]
        ]
      , text caption
      ]
{% endhighlight %}

We can reuse stuff. Let's do the simplest way.

(EXERCISE) Let's refactor the button into a separate function! I bet you can do that! 
(SOLUTION) 

{% highlight elm %}
view model =

  let 
    isDisabled =
      model.quantity <= 0
      
    caption = 
      (toString model.quantity) 
      ++ " "
      ++ (pluralize "shelf" "shelves" model.quantity)   
    
  in
    div [class "content" ]
      [ h1 [] [ text "Pluralizer" ]
      , div [] 
        [ quantityButton "Increase" Increase
        , quantityButton "Decrease" Decrease
        ]
      , text caption
      ]
      
quantityButton caption msg =
  button [ onClick msg ] [text caption ]
{% endhighlight %}

The best way is creating your own modules. But it is a bit out of scope here.

## Type annotation (currying)

So far we have created types and functions and Elm has inferred the types.
Elm is inspired on Haskell and Haskell has something called type annotation.

We could write typical comments, like:

{% highlight elm %}
-- takes two strings and a quantity and returns a string
pluralize singular plural quantity =
  if quantity == 1 then
    singular
  else
    plural
{% endhighlight %}

But we know that gets out of date very quickly, why don't we write something that the compiler will understand.
{% highlight elm %}
pluralize : String -> String -> Int -> String
pluralize singular plural quantity =
  if quantity == 1 then
    singular
  else
    plural
{% endhighlight %}

If we change the types the compiler will tell us. It is a more reliable documentation.
You can look at that as: pluralize takes a String, String and an Int and returns a String.
But it is not totally correct.

Actually with this we can do something really good. Let's say I want to do a function to pluralize shelves:

{% highlight elm %}
pluralizeShelf quantity =
  pluralize "shelve" "shelves" quantity
{% endhighlight %}

The type of that function is:
{% highlight elm %}
pluralizeShelf : Int -> String
  pluralize "shelve" "shelves" quantity
{% endhighlight %}

So the cool thing about currying is that if you call pluralize and you don't pass all arguments is going to give you back another function which takes whatever arguments you had left. Then you can call that other function to sort of finish the job.
For example, if we change pluralizeShelf to the following, it is going to do the same thing:
{% highlight elm %}
pluralizeShelf =
  pluralize "shelve" "shelves"
{% endhighlight %}

Another example:
{% highlight elm %}
pluralizeShelves : Int -> String
pluralizeShelves =
  pluralizeShelf "shelves"
  
pluralizeShelf : String -> Int -> String
pluralizeShelf =
  pluralize "Shelves"
{% endhighlight %}

So pluralizeShelf is going to take the reminder of what pluralize needs. Finally pluralize type notation can be represented as:
{% highlight elm %}
pluralize : String -> String -> (Int -> String)
{% endhighlight %}
{% highlight elm %}
pluralize : String -> (String -> (Int -> String))
{% endhighlight %}

If you take out the parentheses that's exactly what we had in the beginning.

(EXERCISE) Can you write the type annotation of one of the functions we have created? Any will do! 

## Type alias

Type alias allows you to redefine the name of a type:

{% highlight elm %}
type alias Model =
  { quantity : Int }
{% endhighlight %}

So then we can specify this type in our functions:

{% highlight elm %}
update : Msg -> Model -> (Model, Cmd a)
{% endhighlight %}


## The challenge

I know you can do this, let's use all our knowledge to provide in the UI the word that we want to pluralize and the pluralized form. Then the rest of our UI should display one or the other.
We need to add a couple of input fields (singular and plural). This is how you add an input field in Elm:

{% highlight elm %}
input [ type' "text", placeholder "Plural", onInput Plural ] []
{% endhighlight %}

We provide a bunch of attributes, one of them is function onInput, it will send a message Plural in that case. Update function can handle that. You'll have to add that message to the Message type and handling of that in the update (maybe adding some props to the model?).

The new message will have a string parameter with the input, so in the Msg type you can define it:
{% highlight elm %}
type Msg = 
  Increase
  | Decrease
  | Singular String
  | Plural String
{% endhighlight %}

In the update you can use the case form which allows you to do pattern matching on the message:

{% highlight elm %}
update msg model =
  case msg of
    Increase ->
      ({ model | quantity = model.quantity + 1 }, Cmd.none)
    
    Decrease ->
      ({ model | quantity = Basics.max 0 (model.quantity - 1) }, Cmd.none)
      
    Singular newSingular ->
      ({ model | singular = newSingular }, Cmd.none)
      
    Plural newPlural ->
      ({ model | plural = newPlural }, Cmd.none)
{% endhighlight %}

Finally you may want to change the pluralize function. For convenience you may want to just use pluralize and pass the model parameters to it.

This is my complete solution:

{% highlight elm %}
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Html.App as Html


init = 
  ({ quantity = 5, singular = "shelf", plural = "shelves"  }, Cmd.none)
  
type Msg = 
  Increase
  | Decrease
  | Singular String
  | Plural String

type alias Model =
  { quantity : Int
  , singular: String
  , plural: String
  }
    
update : Msg -> Model -> (Model, Cmd a)
update msg model =
  case msg of
    Increase ->
      ({ model | quantity = model.quantity + 1 }, Cmd.none)
    
    Decrease ->
      ({ model | quantity = Basics.max 0 (model.quantity - 1) }, Cmd.none)
      
    Singular newSingular ->
      ({ model | singular = newSingular }, Cmd.none)
      
    Plural newPlural ->
      ({ model | plural = newPlural }, Cmd.none)
  
subscriptions model =
  Sub.none
  
pluralize : String -> String -> Int -> String
pluralize singular plural quantity =
  if quantity == 1 then
    singular
  else
    plural

view model =

  let 
    isDisabled =
      model.quantity <= 0
      
    caption = 
      (toString model.quantity) 
      ++ " "
      ++ (pluralize model.singular model.plural model.quantity)   
    
  in
    div [class "content" ]
      [ h1 [] [ text "Pluralizer" ]
      , div []
        [ input [ type' "text", placeholder "Singular", onInput Singular ] []
        , input [ type' "text", placeholder "Plural", onInput Plural ] []
        ]
      , div [] 
        [ quantityButton "Increase" Increase False
        , quantityButton "Decrease" Decrease isDisabled
        ]
      , text caption
      ]
      
quantityButton caption msg isDisabled =
  button [ onClick msg, disabled isDisabled ] [text caption ]
    
main = Html.program
    { init = init,
    view = view,
    update = update,
    subscriptions = subscriptions
    }
{% endhighlight %}

Please, shout out your questions, issues, concerns, insults, matters, you can swear.

## Some resources to go on with the learning

[Good document, like a book, loads of examples](https://www.gitbook.com/book/evancz/an-introduction-to-elm/details)

[One  intro blogpost with useful links](https://pragmaticstudio.com/blog/2014/12/19/getting-started-with-elm)

[Code examples](http://elm-lang.org/examples)

[Practical example of implementing Elm](https://gist.github.com/ohanhi/0d3d83cf3f0d7bbea9db)

[Elm tests](https://github.com/deadfoxygrandpa/Elm-Test)

[Video turorials](https://www.dailydrip.com/topics/elm)

[Implementing a whole app step by step](https://www.gitbook.com/book/sporto/elm-tutorial/details)

[Learning Elm for JS developers](https://github.com/elm-guides/elm-for-js)

[Podcast with Elm creator Evan Czaplicki](https://devchat.tv/js-jabber/175-jsj-elm-with-evan-czaplicki-and-richard-feldman)

[Introducing Elm in a real web app in a company](http://tech.noredink.com/post/126978281075/walkthrough-introducing-elm-to-a-js-web-app)

[This post](https://github.com/javflores/javflores.github.io/blob/master/_posts/Elm-workshop.md)

[My Elm examples](https://github.com/javflores/elm-playground)

