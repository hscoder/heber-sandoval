---
layout: post
title:  "Learning Sinatra"
date:   2016-05-03
categories: Sinatra
---

## Introduction to Sinatra

Sinatra is a Domain Specific Language implemented in Ruby that's used for writing web application. To install Sinatra, run the following command in your terminal:

```
gem install sinatra
```

Any application that requires the `sinatra` library will have access to methods like: `get` and `post`. As a result, these methods will give you the ability to transform a Ruby application into an application that can respond to HTTP requests.

My first Sinatra app, was the usual ["Hello, World"](https://github.com/hscoder/sinatra-hello-world-basics-v-000) example.

To run this application in your browser, type `shotgun` in the terminal to start the server.

{% highlight ruby %}
require 'sinatra'

class App < Sinatra::Base
  get '/' do
    "Hello, World!"
  end
end
{% endhighlight %}

If you visit `http://localhost:9292/` in your browser, the following text will be displayed:

```
Hello, World!
```

### Routes

Routes are the part of an application that **connects** HTTP requests to a specific method in your application code built to handle responding to such a request (that part of code is called a Controller Action).

When a user clicks a link on a webpage, they are making a **HTTP request** to get a specific URL. How is that request handled? This is done by pairing an HTTP method/verb, like `GET` or `POST` with an URL matching pattern. This usually matches what the user typed in the browser.

##### How it works

{% highlight ruby %}
get '/posts' do       # this `GET` request will trigger when navigating to `http://localhost:9292/posts`.
  @posts = Post.all   # the controller will ask the `Post` model to find all posts and store them in `@posts`.
  erb :'posts/index'  # the view can render all the posts via the `@posts` variable.
end
{% endhighlight %}

The `erb :'posts/index'` code snippet from above, will use the instance variable via `erb tags`. This is just **Embedded Ruby** mixed with HTML, here's a sample:

{% highlight erb %}
<h1>View all your Posts</h1>
<% @posts.each do |post| %>
  <h4>Author: <%= post.user.username %></h4>
  <h4>Title: <a href="/posts/<%=post.id%>"><%=post.title%></a></h4>
  <p>Content: <%=post.content%></p>
<% end %>
{% endhighlight %}

In this example, anything between `<%=  %>` or `<%  %>`, is interpreted as Ruby code. Here, we're iterating over each posts and retrieving the value of the username's attribute along with the id, title and content attribute.

##### Overview

Here is the file structure of a simple blog application.

```
.
├── app
│   ├── controllers
│   │   ├── application_controller.rb
│   │   ├── posts_controller.rb
│   │   └── users_controller.rb
│   ├── models
│   │   ├── post.rb
│   │   └── user.rb
│   └── views
│       ├── index.erb
│       ├── layout.erb
│       ├── posts
│       │   ├── edit.erb
│       │   ├── index.erb
│       │   ├── new.erb
│       │   └── show.erb
│       └── users
│           ├── login.erb
│           └── signup.erb
├── config
│   └── environment.rb
├── config.ru
├── db
│   ├── development.sqlite
│   ├── migrate
│   │   ├── 20160505202627_create_users.rb
│   │   └── 20160505202648_create_posts.rb
│   └── schema.rb
├── Gemfile
├── Gemfile.lock
├── Rakefile
├── README.md
└── spec
    └── spec_helper.rb
```
This gives us an idea of who is doing what.

### Forms

A form is the most common way for users to pass data to a web application.

{% highlight html %}
<h1>Enter a New Post</h1>
<form action="/posts" method="POST">
  <label>Title: <input type="text" name="title" id="title"></label>
  <br>
  <label>Post: <input type="text" name="content" id="content"></label>
  <br>
  <input type="submit" value="Submit">
</form>
{% endhighlight %}

This form has a `Submit` button, that once clicked, its sends a `POST` request. This request will be routed to the corresponding route in the `posts_controller` action that matches the form's `action="/posts"` and `method="POST"` attributes.

{% highlight ruby %}
post '/posts' do
  if params[:content] == ""
    redirect to '/posts/new', locals: {message: "Please fill in the Content."}
  else
    @user = User.find_by_id(session[:user_id])
    @post = Post.create(content: params[:content], title: params[:title], user_id: @user.id)
    redirect to "/posts/#{@post.id}", locals: {message: "Post successfully saved!"}
  end
end
{% endhighlight %}

After some validation, ActiveRecord preforms a query, `.find_by_id`, which takes an argument of `session[:user_id]`. This gets the current user's id fom the session hash. Then, with the data the user typed in the form, the controller asked the `Post` model to `.create` a post using the params hash values, as seen above. Once the post have been persisted, the user will be redirect to `"post/#{@post.id}"`, which will show or display their post.

### Cookies Sessions

A **cookie** is effectively just storing a hash on the browser that then gets sent with every request. A **session** is simply an object, like a hash, that stores data describing a client's interaction with a website at a given point in time.

### ActiveRecord

Sinatra doesn't come with database support out of the box. In oder to use the functionality of ActiveRecord, simply include it in your Gemfile, among other supporting gems.

```
source "https://rubygems.org"

gem 'sinatra'
gem 'activerecord', :require => 'active_record'
gem 'sinatra-activerecord', :require => 'sinatra/activerecord'
gem 'rake'
gem 'require_all'
gem 'sqlite3'
gem 'thin'
gem 'shotgun'
gem 'pry'
gem 'bcrypt'
gem "tux"
gem "rack_session_access"

group :test do
  gem 'rspec'
  gem 'capybara'
  gem 'rack-test'
  gem 'database_cleaner', git: 'https://github.com/bmabey/database_cleaner.git'
end
```
You now have access to database support.

```
# config/environment.rb
ENV['SINATRA_ENV'] ||= "development"

require 'bundler/setup'
Bundler.require(:default, ENV['SINATRA_ENV'])

ActiveRecord::Base.establish_connection(
  :adapter => "sqlite3",
  :database => "db/#{ENV['SINATRA_ENV']}.sqlite"
)

require_all 'app'
```
 ... and you can create Rake tasks.

```
heber@heber:~/code/labs/blog$ rake -T
rake db:create              # Creates the database from DATABASE_URL or con...
rake db:create_migration    # Create a migration (parameters: NAME, VERSION)
rake db:drop                # Drops the database from DATABASE_URL or confi...
rake db:fixtures:load       # Load fixtures into the current environment's ...
rake db:migrate             # Migrate the database (options: VERSION=x, VER...
rake db:migrate:status      # Display status of migrations
rake db:rollback            # Rolls the schema back to the previous version...
rake db:schema:cache:clear  # Clear a db/schema_cache.dump file
rake db:schema:cache:dump   # Create a db/schema_cache.dump file
rake db:schema:dump         # Create a db/schema.rb file that is portable a...
rake db:schema:load         # Load a schema.rb file into the database
rake db:seed                # Load the seed data from db/seeds.rb
rake db:setup               # Create the database, load the schema, and ini...
rake db:structure:dump      # Dump the database structure to db/structure.sql
rake db:structure:load      # Recreate the databases from the structure.sql...
rake db:version             # Retrieves the current schema version number
```
This allows you to create a db table by running `rake db:create_migration NAME=create_posts`.

{% highlight ruby %}
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.string :content
      t.integer :user_id
    end
  end
end
{% endhighlight %}

Run `rake db:migrate` to make the table with its columns, as shown above.

### Connecting Controller Action to Views
```
              Create
| get '/posts/new' | post '/posts'|
| :--------------- | :----------- |
| new.erb          | new.erb      |

               Read
| get '/posts'     | get '/posts/:id'|
| :--------------- | :-------------- |
| index.erb        | show.erb        |

                  Update
| get '/posts/:id/edit' | post '/posts/:id'|
| :-------------------- | :--------------- |
| edit.erb              | edit.erb         |

Delete
| get '/posts/:id' | post '/posts/:id/delete'|
| :--------------- | :---------------------- |
| show.erb         | show.erb                | *this page contains the delete button that post to ...
```
More to come.

Great!
