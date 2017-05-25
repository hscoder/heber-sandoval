---
layout: post
title:  "Rails Project"
date:   2016-09-01
categories: Rails
---

## Brief Overview of Rails Project - Blogr

For this project, I'll be building a simple blog application that a user can make a post with categories. Also, other users can comment on other user's posts. This app will have a simple authentication system, nested routes, nested forms along with basic style. It's a good idea to have clear picture of your app's final result. Though some features may be added or omitted, this can help you keep on track.

First, we need to set up our models, migrations, resources, controllers, views and anything else our app might need. We'll start one model at a time.

Here's a birds view of the model structure:
table


#### users table
```
| id  | name | email |
| :-- | :--- | :-----|
|     |      |       |
```

#### posts table
```
| id  | title | body | user_id | category_id |
| :-- | :---- | :--- | :------ | :-----------|
|     |       |      |         |             |
```

#### categories table
```
| id  | name |
| :-- | :----|
```

#### comments table
```
| id  | body  | post_id  | user_id |
| :-- | :-----| :------- | :-------|
|     |       |          |         |
```

```
users
 - id
 - name
 - email

posts
 - id
 - title
 - body
 - user_id
 - category_id

 categories [*?* a category has_many :comments through: posts *?*]
  - id
  - name

 comments
  - id
  - body
  - post_id
  - user_id
```

Lets get started!

## Creating the User Model

#### users table
```
| id  | name | email |
| :-- | :--- | :-----|
|     |      |       |
```

 To start a new project, run:

 `$ rails _4.2.5_ new blogr`

 then,

 `$ cd blogr`

 Run `rails server` and if all works, you should see the generic "Welcome to Rails" page.

 To begin...

 Generate the User model, run this command from the terminal:

`$ rails g resources User name email`

Running this code generates several files inside our app directory. Under the `db` directory, we find a directory called `migrate` that contains the db migration. Inside that directory, a file called, `create_users.rb` contains this code:

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email

      t.timestamps null: false
    end
  end
end
```

Run migration: `rake db:migrate`. This creates a table named `users` with fields named `id`, `name`, `email`, `created_at` and `updated_at`.

The `name` and `email` fields will store data entered by the user. Rails adds the `id`, `user_id`, `category_id`, `created_at` and `updated_at` fields automatically. Although not shown in the above code, the `id` field is a unique, auto-incrementing integer that represents each row in the database. The `created_at` and `updated_at` are timestamps representing when the row was created and when it was last updated.

Code in `app/models/user.rb`

```ruby
class User < ActiveRecord::Base
end
  ```
Code in `app/controllers/users_controllers.rb`

```ruby
class UsersController < ApplicationController
end
```

## Creating the Post Model

#### posts table

```
| id  | title | body | user_id | category_id |
| :-- | :---- | :--- | :------ | :-----------|
|     |       |      |         |             |
```

Brief description goes here.

`$ rails g resources Post title body:text references:user references:category`

This code generates several files inside our app directory. Under the `db` directory, we find a directory called `migrate` that contains the db migration. Inside that directory, a file called, `create_posts.rb` contains this code:

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.text :body
      t.references :user, index: true, foreign_key: true
      t.references :category, index: true,
      foreign_key: true

      t.timestamps null: false
    end
  end
end
```

Running this migration from the terminal with `rake db:migrate`, creates a table named `posts` with fields named `id`, `title`, `body`, `user_id`, `category_id`, `created_at` and `updated_at`.

The `title` and `body` fields will store data entered by the user. Rails adds the `id`, `user_id`, `category_id`, `created_at` and `updated_at` fields automatically. Although not shown in the above code, the `id` field is a unique, auto-incrementing integer that represents each row in the database. The `created_at` and `updated_at` are timestamps representing when the row was created and when it was last updated.

So far, the app doesn't have much code.

Code in `app/models/post.rb`

```ruby
class Post < ActiveRecord::Base
end
  ```
Code in `app/controllers/posts_controllers.rb`

```ruby
class PostsController < ApplicationController
end
```
