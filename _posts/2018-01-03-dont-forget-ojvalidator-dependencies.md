---
layout: post

title: "Don't Forget Your ojColorConverter Dependencies"

excerpt: "When I try to convert JET colors, I never seem to get the dependencies right on the first try.  Plus I forget how to setup the converterFactory and pass in the right formats."

tags: [Oracle JET, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

I mess this up every time I try to use it:

If you're attempting to leverage the [Oracle JET color converter](http://www.oracle.com/webfolder/technetwork/jet/jsdocs/oj.ColorConverter.html), you'll get these errors in your browser console if you forget the dependencies at the top of your ojModule's `define()` function:

<div class="highlight"><pre><code class="language-bash" data-lang="bash"><span style="color: red;">TypeError: Cannot read property 'converterFactory' of undefined</span>
    // you left out 'ojs/ojvalidation'
</code></pre></div>

or

<div class="highlight"><pre><code class="language-bash" data-lang="bash"><span style="color: red;">TypeError: oj.Color is not a constructor </span>
    // you left out 'ojs/ojcolor'
</code></pre></div>

Also, besides including  `'ojs/ojvalidation'` and `'ojs/ojcolor'`, watch to make sure you perform all the steps:

1. Create a ConverterFactory of type color
2. Pass in an options object for the format you want to output and create the converter itself
3. Make sure the input color is an ojColor object first (see below for a trick for using [CSS Color Keywords](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value#Color_keywords) directly)
4. Run it through the converter

{% highlight js %}

define(['ojs/ojcore', 'knockout', 'jquery', 'ojs/ojvalidation', 'ojs/ojcolor'],
  function(oj, ko, $) {
  
    ...

    // Steps 1 & 2 (one time only) 
    // Use a factory to create a converter for the output you want 
    var cf = oj.Validation.converterFactory(oj.ConverterFactory.CONVERTER_TYPE_COLOR);
    var convHsl = cf.createConverter({"format": "hsl"}) ;


    // Steps 3 & 4
    // Create a Color object first, then convert it
    var rgbaOrig = new oj.Color("rgba(27,128,254,0.8)");
    var convertedRgbaColor = convHsl.format(rgbaOrig);  
        // returns "hsla(213, 99%, 55%, 0.8)"

    var hexOrig = new oj.Color("#fa8072");
    var convertedHexColor = convHsl.format(hexOrig);
        // returns "hsl(6, 93%, 71%)" 

    var objectOrig =  new oj.Color({h:310, s:50, v:80, a:0.8});
    var convertedObjectColor = convHsl.format(objectOrig); 
        // returns "hsla(310, 50%, 60%, 0.8)"



    // What if you want to use CSS3 color specification strings?
    // Convert it in one fell swoop using object bracket notation
    //  (but be advised the CSS3 color properties are in ALLCAPS 
    //   on the oj.Color object): 
    var convertedCSS3str = convHsl.format(oj.Color['deeppink'.toUpperCase()]);          // returns "hsl(328, 100%, 54%)"



    ...

{% endhighlight %}


Here's a little fun with converters as a bonus:


<iframe style="margin-left: -40px; margin-bottom: 50px; width: 680px;" height="600" src="//jsfiddle.net/pthaden/pp8fw4ew/embedded/result,js,html,css/" allowpaymentrequest allowfullscreen="allowfullscreen" frameborder="0"></iframe>

