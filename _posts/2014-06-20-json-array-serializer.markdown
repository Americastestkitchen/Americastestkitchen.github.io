---
layout: post
title:  "JSON Array Serializer"
github: Americastestkitchen/json_array_serializer
author: Nathan Lilienthal
github_username: nixpulvis
date:   2014-06-15 23:15:00
categories: rails
---

Saving things to the database in Rails is generally very easy. It handles strings, numbers, booleans, and other common types pretty well. One common data type that is not as well supported is JSON arrays. Whenever you have a collection of objects, be it an `Array` of `Hash` or something custom like a `DeveloperArray` of `Developer`, saving it in Rails can be a pain. What would be nice is to have a way to build on to Rail's serializers to make it possible to save and load these kinds of data transparently like everything else.

Enter [JSONArraySerializer](https://github.com/Americastestkitchen/json_array_serializer).

{% highlight ruby %}
class Developer < OpenStruct
  def develop(name)
    # Write code.
  end
end

class DeveloperArray < Array
  def develop
    each { |d| d.develop }
  end
end

class Company < ActiveRecord::Base
  serialize :developers, JSONArraySerializer.new(array_class: DeveloperArray, element_class: Developer)
end
{% endhighlight %}

## Specifying a Common Interface

The first thing we need to do is declare the interface. What **must** every object know how to do in order to be serialized. First of all we have two types of objects to deal with, the array object, and the element object.

### Array Class API

From the docs:

{% highlight ruby %}
A.new : Array -> A
a.to_a : -> Array (where a is an instance of A)
{% endhighlight %}

So, for example if we had `DeveloperArray.new([Developer.new, Developer.new])` then the `DeveloperArray` class would need it's `.new` method to accept 1 `Array` argument and return an instance of itself. It also needs to define a `#to_a` method which returns an array

### Element Class API

From the docs:

{% highlight ruby %}
A.new : Hash -> A
a.to_h : -> Hash (where a is an instance of A)
{% endhighlight %}

Much like the array class, this is just saying that the `Developer` class must have a `.new` method taking a `Hash` and returning an instance of itself, and a `#to_h` method which returns a `Hash`.

### The Serializer API

This interface is very simple, and conveniently already implemented by `Array` and `Hash`, making them the perfect defaults for our library. Making use of custom classes should be just a matter of specifying them when we create our serializer.

The next decision we need to make is where to put the class name information. Clearly we expect the a `DeveloperArray` to be saved and loaded back into another `DeveloperArray`, but how will Rails know to do that? YAML solves this problem by saving class names in the data representation itself. I opted to have the information saved in the serializer itself. The advantages of this are twofold.

First, the data will be pure, clean JSON. There will not be any ruby specific attributes injected, so loading these columns from the database will be simple and portable.

Second, it makes our implementation simpler. We don't need to do any evaluation of the data before we start putting it into the right classes. The serializer knows what array class and element class to put things in already.

Creating a serializer is simple.

{% highlight ruby %}
# .new -> JSONArraySerializer
# #load : [JSON String] || JSON String -> array_class<element_class>
# #dump : array_class<element_class> -> [JSON String] || JSON String

JSONArraySerializer.new(array_class: DeveloperArray, element_class: Developer)
{% endhighlight %}

> Ok, what's with the method signatures for `#load` and `#dump`? Why do they deal with either an array of strings *or* just a string.

This is to support multiple column types. All databases have a notion of text, but some have a notion of an array. Rails when saving and loading from each loads them into the expected datatypes `String` and `Array` respectively. For our code to work we need to know what type of column is backing the data. the `#load` and `#dump` methods handle their data based on this option.

From the docs:

{% highlight ruby %}
 serialize :foo, JSONArraySerializer.new                       # Will save to :text.
serialize :foo, JSONArraySerializer.new(column_type: :text)   # Same as above.
serialize :foo, JSONArraySerializer.new(column_type: :array)  # Saves to array column.
{% endhighlight %}

---

This gem, while simple, has allowed us to DRY up various parts of our code. Areas that before had their own custom serializers which were all doing almost the same things. Making one implantation for all our use cases allowed me to write tests which cover everything in one central place. This improves overall test coverage and therefor improves confidence. When dealing with things like saving credit card token arrays, or access arrays, confidence is the most important commodity you can have.
