---
layout: post
title:  "Learning Sinatra"
date:   2016-05-03
categories: Sinatra
---

### Introduction to Sinatra

Sinatra is a Domain Specific Language implemented in Ruby that's used for writing web application. To install Sinatra, run the following command in your terminal:

```
gem install sinatra
```

My first Sinatra app, was the usual ["Hello, World"](https://github.com/hscoder/sinatra-hello-world-basics-v-000) example.

{% highlight ruby %}
require 'sinatra'

class App < Sinatra::Base

  get '/' do
    "Hello, World!"
  end

end

#=> going to http://localhost:9393/

#=> displays 'Hello, World!' in the browser.
{% endhighlight %}

#### Routes

Write something here

#### Forms

A basic example will do.

#### Sessions

Something about sessions.

#### ActiveRecord

Something about ActiveRecord.

Great!
