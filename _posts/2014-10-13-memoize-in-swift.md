---
layout: post
title: Memoize in Swift
description: An explanation of a memoization implementation in swift
image: assets/images/swift.jpg
---

I've been learning swift lately in my IOS class and was introduced to closures. After working on the [memoize]({{site.baseurl}}{% post_url 2014-10-12-memoize %}) code in javascript a few days ago I realized that I could implement a similar solution in swift.  There are a few caveats though because of differences in the language and flexibility.

So here is the memoize function in swift:

{% highlight swift %}
func memoize(function:([Any]) -> Any?) -> (Any...) -> Any? {
  var cache = [String: Any]()

  return {(parameters:Any...) -> Any? in
    var key:String = ""
    if parameters.count > 0 {
      key = parameters.description
    }

    if (cache[key] != nil) {
      return cache[key]
    } else {
      var value = function(parameters)
      return value
    }
  }
}
{% endhighlight %}
                               
So here is an explanation of the code.  If you are familiar with my javascript solution, this will look very similar but there are a few differences so I will focus on those differences.

First of all the function signature is much different because of the pseudo dynamic types in swift.  Although variables can be typed at compile time, it will eventually get typed, and once set, it can't change.

So lets break down the signature.

{% highlight swift %}
func memoize(function:([Any]) -> Any?) -> (Any...) -> Any?
{% endhighlight %}

So the memoize function has a single parameter, a closure that takes an array of Any and returns an Any optional.  The reason why it taks an array of Any will be more clear as I go further into the function.

The memoize function will return a closure of type Any variadic that returns an Any optional.  The reason why the returned function takes an Any variadic is so that the function can be called with as many parameters as necessary.

For those unfamiliar, variadic parameters allow a function to take an unlimited number of parameters of the given type.  So a function with a variadic parameter could be called like `foo(1)` or `foo(1, 2, 3)`. All of the parameters will be in an array of the parameter name.  For more info on variadic parameters checkout [Apples documentation](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html).

The rest of the memoize function is pretty much the same as the javascript implementation except for the calling of the original function.  I couldn't find a way to call a function in swift like you can with the javascript apply function so we just pass in the array of parameters catpured in the variadic parameter.  This is why the closure that the memoize function takes must have an input parameter of type [Any] because all of the parameters will be passed into it as an array.  This is less than ideal but is the only solution that I could find.

So that's the just of the memoize function.  In order to use the function, the closure you write must be a specific type.

Here is an example of a possible function that could be memoized.

{% highlight swift %}
var sum = {(parameters:[Any]) -> Any? in
    var result:Int = 0
    for num in parameters {
          result += num as Int
    }
    return result
}
{% endhighlight %}
    
Then we would memoize it by passing it to memoize.  But because the closure that is returned from memoize is of a different type than the closure we are passing into memoize, we must assign the result to a new variable.

{% highlight swift %}
var memSum = memoize(sum)
{% endhighlight %} 

So now if we run memSum:

{% highlight swift %}
memSum(1, 2) //returns 3 from the original function
memSum(1, 2) //returns 3 from the cache
{% endhighlight %}  

So there you have it.  A memoize function in swift.  Although it may not be ideal to write your closures with the required type, it could prove beneficial at some point.
				