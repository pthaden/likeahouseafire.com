---
layout: post

title: "How Do You Pronounce \"Fat&nbsp;Arrow?\""

excerpt: "I see it everywhere these days, but how do you *say* it?"

tags: [JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Do you have a little voice in your head that vocalizes the things you read? What do you say when you see the `=>` symbol?

I've been playing around a bit with ES6 features, including the new ["fat arrow" syntax for anonymous functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).  Lots of Node.js code is written in this style, as are almost all of the Functional Programing tutorials and examples.  It looks something like this:

{% highlight js %}

(x, y) => x * y;

{% endhighlight %}

as opposed to this:


{% highlight js %}

function multiply(x, y) {
  return x * y;
}

{% endhighlight %}

I'll leave it to smarter people to describe all the nuances of the fat arrow syntax, like how convenient it is to pass their shorter syntax in to higher-order functions like `map`, `filter` and `reduce`, how they don't bind their own `this` or have an `arguments` object, and how they're much easier to read when implementing functional-style ideas like currying and such.

Meanwhile, I realize just because they're new and shiny doesn't make them perfect, as [Kyle Simpson points out in a decision flowchart from *You Dont Know JS: ES6 & Beyond:*](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20&%20beyond/fig1.png)

<div class="full zoomable"><img src="https://raw.githubusercontent.com/getify/You-Dont-Know-JS/master/es6%20%26%20beyond/fig1.png"></div>




## About that voice in my head

So when I'm reading code samples to myself, I'm often unsure if I'm using the right word for a particular symbol. Take the equals sign (`=`) for example. Depending on how many there are, I read the code and say a different phrase in my head:  

|**Example symbol&nbsp;usage**|**JavaScript meaning**|**Voice in my head**|
|`var x = 1;`|assignment|"variable x is assigned one"|
|`if (x == "1")...`|loose equality|"if x is equal to one"|
|`if (x === 1)...`|strict equality|"if x is really, *really* equal to the number one"|


<br>Now take the function declaration at the top of the article.  Here's how I'd voice that when reading it to myself:

|**Code**|**Voice in my head**|
|`function multiply(x, y) {`<br>&nbsp;&nbsp;`return x * y;`<br>`}`|"function 'multiply' takes an x and a y and returns x times y"|


<br>My problem:  I don't know what word to use when reading the Fat Arrow syntax:

|**Code**|**Voice in my head**|
|`let multiply = (x, y) => x * y;`|"let 'multiply' be assigned a function that takes an x and a y and...returns?...yields?...maps to?...fat-arrows-on-into?...umm?..."|


##What people are saying

The [aforelinked MDN article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) gives a great discussion on the new syntax, but never actually tells you how it's read properly.

So I've been on a mission to find the right way to say JavaScript's `=>` symbol out loud.

Other languages have had arrow functions for a while, and someone asked [How do I pronounce “=>” as used in lambda expressions in .Net](http://stackoverflow.com/questions/274022/how-do-i-pronounce-as-used-in-lambda-expressions-in-net). Answers came back as 'such that', 'goes to', 'becomes', 'lambda-of', 'maps-to' and simply 'to'.

Slightly related to where I think the fat arrow is coming from, Wikipedia's [Lambda Calculus article](https://en.wikipedia.org/wiki/Lambda_calculus#Motivation) parenthetically says a bit of MathML should be "read as 'the pair of x and y *is mapped to* x^2 + y^2'."  Which then if you look at the symbols embedded in that article and then start googling around, winds you back at Wikipedia at an [article with nothing but the symbol "→" in its URL](https://en.wikipedia.org/wiki/→).

Douglas Crockford doesn't give a pronunciation for the symbol, but has decided to [name fat arrow functions "farts"](https://plus.google.com/+DouglasCrockfordEsq/posts/TxQ4gRkZxST) and [`=>` the fart operator](http://jslint.com/help.html).

Meanwhile, I stumbled on a [video where James Coglan says](https://youtu.be/XcS-LdEBUkE?t=4m18s) CoffeeScript's thin-arrow symbol is read as 'takes...and returns' but also can be read as 'from...to' when he says: "map takes a function from [a] to [b]... and returns a list of [b]'s".

And the response to the question ["How to 'read' arrow functions in ES6?"](http://softwareengineering.stackexchange.com/questions/324656/how-to-read-arrow-functions-in-es6) gets oh-so-close, but when it says "and returns" seems to me to be talking about the structure of the function (especially the old ES5 style) and not the way you'd read the code itself.

Have you heard an official way to pronounce `=>` when reading JavaScript code?  How do you say it?







