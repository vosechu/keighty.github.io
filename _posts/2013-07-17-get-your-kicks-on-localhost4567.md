---
layout: post
title: "Get your kicks on route localhost:4567"
description: ""
category: Sinatra
tags: []
---
{% include JB/setup %}
##I learned something about Sinatra today.
A route has three ingredients: an HTTP method, a URL matching pattern, and a code block.
Typically they look like this:
{% highlight ruby %}
get '/' do
  "hello world"
end{% endhighlight %}

But they can also look like this:
{% highlight ruby %}
post '/game/:guess' do
  game.guess(params[:guess])
end
private
  def game
    @game ||= Mastermind::Game.new_with_code(params[:code])
  end{% endhighlight %}

The code block returns information. That information can be a web page, basic text (first example), or a the result of an evaluated block of code (second example). The block ships the return value back to the browser that invoked it. What the browser does with it is the browser's business.

##I learned something about Ajax today.
An ajax call is built up from a few minimum parameters: a url, the HTTP method, the dataType expected, and a callback function that handles the return. You can include data from the user in the form of a data hash:

{% highlight javascript linenos%}
var the_url = "http://" + window.location.host + "/game/" + guess_string;
  var mark_string = $.ajax({
    type: "POST",
    url: the_url,
    accepts: "application/json",
    dataType: "json",
    data: { 'code' : secret_code },
    complete: function(data){
      output_mark = data['responseText'];
      process_output(secret_code, guess_string, output_mark);
    }
  });{% endhighlight %}

In this code snippet I am using Ajax to send the secret code and the guess to the Sinatra route . I let the route do its magic, and wait for it to send me back a json containing the result.

The overall workflow looks like this:
1. The url will look like this: "http://localhost:4567/game/12345" (line 1). The submitted guess is 12345.
2. Ajax packages the url with the data hash containing the secret code (line 7).
3. Javascript submits the HTTP POST request to Sinatra which matches the route above.
4. The route creates a Mastermind::Game object with the provided secret code.
5. The route calls the guess method on the Game object with the submitted guess.
6. The route returns a json of the method output (in this case it is a string of +/- characters indicating exact number matches(+) or matched numbers in the wrong position(-)).
7. In the ajax callback function the data from the json object is parsed and passed on for further processing.

##I learned something about routes today.
I was experiencing a mental block about routes. I thought that building the "/game/:guess" route in Sinatra meant that I would be constantly navigating between "/mastermind" and "/game/:guess", and that I would always be reloading the game with a new secret. How could my user interact with a page that was constantly reloading? But I was thinking about routes the wrong way. When my javascript called the route, it wasn't reloading my page, it was just retrieving data! I had conflated routes with navigation in my imagination, and forgot that a route is just a request for information, or a method call. Just because the information returned is usually rendered as an html page doesn't mean that it has to be html. Which leads me to the penultimate point...

##I learned something about curl today.
ANYTHING can make an HTTP request. Ok, maybe that is a slight overstatement.

Browsers interpret the information retrieved using HTTP requests, but they are far from the only category of software that can. Turns out, the command-line functions just as well:
{% highlight bash linenos%}
$ curl -X POST -H "Accept: application/json" -d "code=12345" js-games.herokuapp.coe/12345
+++++${% endhighlight %}

The command on line 1 will return the same string I am retrieving with $.ajax in javascript (output on line2). Sure it is a fairly plain result -- it doesn't even include a new line character, for pity's sake -- but it does the job of making an HTTP request to a route and receiving the result of the route's block processing.

##I learned something about myself today.
I didn't even realize I was misunderstanding HTTP! It was one of those concepts I had already checked off my mental list of technologies to cover on my quest to become a web developer. This breakthrough has reminded me that reviewing the fundamentals in the context of learning new technologies will bring a deeper understanding of everything.

Awesome