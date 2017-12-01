---
title: Sorting multidimensional arrays in Ruby
description: Sorting multidimensional works with a few caveats
image: /assets/images/ruby.jpg
layout: post
---

Recently at work I've been doing some work on a project written in Ruby to enable an endpoint to paginate. Something we need for our mobile apps so we can make a list more performant. In the process of making this work I ran into a problem with trying to sort a multi-dimensional array. Ruby will look at each index in the nested arrays and compare their indexes to determine which array should come first.

{% highlight ruby %}
my_array = [
  [9, 8, 7],
  [6, 5, 4],
  [3, 2, 1]
]

my_array.sort!

# [
#   [3, 2, 1],
#   [6, 5, 4],
#   [9, 8, 7]
# ]
{% endhighlight %}

When we sort the array, it will not sort any nested array items, only use them to determine their ordering in the array being sorted. It does this by using the `<=>` spaceship operator. This operator will return -1, 0, or 1 and works out of the box for arrays. If the first index in both nested arrays is the same, it will move on to the next items to compare.

{% highlight ruby %}
my_array = [
  [2, 1],
  [2, 0]
]

my_array.sort!

# [
#   [2, 0],
#   [2, 1]
# ]
{% endhighlight %}

This is a great productivity win since we don't have to write a custom comparison method to handle multi-dimensional arrays. But there are a few caveats. When comparing to values they must be comparable. For example `1 <=> '1'` are not comparable in ruby. This will return nil. You must ensure that the types of items being compared are comparable using the spaceship operator.

{% highlight ruby %}
my_array = [
  [2, '1'],
  [2, '0']
]

my_array.sort!

# [
#   [2, '0'],
#   [2, '1']
# ]
{% endhighlight %}

Which surfaces another tough problem. The case where items in your array can be nil. If one of your arrays contains nil, the containing array cannot be compared to another array.

{% highlight ruby %}
my_array = [
  [nil, 1],
  [2, 3]
]

my_array.sort!

# Argument Error: comparison of array with array failed
{% endhighlight %}

In this case we have to write our own sorting function that can handle cases with nil. If you know the dimensions of your arrays then it can be fairly straightforward, but when you don't it requires a little more work. In my case I knew that the array I was sorting would only contain other arrays, but beyond that the dimensions were unknown. Here is the solution I came up with.

{% highlight ruby %}
nil_compare = Proc.new do |a,b|
  length = [a.size, b.size].min
  compare_value = 0
  length.times { |i|
    if a[i].nil? && b[i].nil?
      next
    elsif a[i].nil?
      compare_value = 1
    elsif b[i].nil?
      compare_value = -1
    elsif a[i].kind_of?(Array) && b[i].kind_of?(Array)
      compare_value = nil_compare.call(a[i], b[i])
    else
      compare_value = a[i] <=> b[i]
    end

    if compare_value != 0
      break
    end
  }
  compare_value
end

my_array = [
  [nil, 1],
  [2, 3]
]

my_array.sort! &nil_compare

# [
#   [2, 3],
#   [nil, 1]
# ]
{% endhighlight %}

Hopefully this can be helpful for others, as when I was googling for a solution, the only results were for sorting an array where certain items in the array are nil. Where as I needed something for when nested arrays had nil items.