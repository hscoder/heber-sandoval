---
layout: post
title:  "Working with Rails Nested Forms"
date:   2016-10-20
categories: Rails
---

## Working with Rails Nested Forms

Getting nested forms to work correctly

In this brief post, I'll attempt to nest the `Ingredient` model under the `Recipe` model in a form so that a user can input a recipe and its associated ingredients.

To get all the pieces connected, I needed to associate all the models for this exercise, see my blog on "model associations" (create link). Here is the `RecipesController`, with a `new`, `create` action.

```ruby
class RecipesController < ApplicationController
  def new
    @recipe = Recipe.new
  end

  def create
    # raise params.inspect
    @recipe = Recipe.new(recipe_params)
    if @recipe.save
      redirect_to recipe_path(@recipe)
    else
      render :new
    end
  end

  private

  def recipe_params
    params.require(:recipe).permit(:name)
  end
end
```

Below, we have a `form_for` form builder to create a form for a recipe. The part that we'll be concentrating is how to associate any amount of ingredients to this recipe. For now, this is being hard-coded in html.

```html
<h1>New Recipe</h1>

<%= form_for(@recipe) do |f| %>
  <%= f.label :name %>
  <%= f.text_field :name %>

  <p>Add Ingredients:</p>
  <ul>
    <li><input type="text" name="ingredients[][name]"></li>
    <li><input type="text" name="ingredients[][name]"></li>
    <li><input type="text" name="ingredients[][name]"></li>
  </ul>

  <%= f.submit %>
<% end %>
```

Right now, we can grab the value of `params` in the form of an array of hashes, which we hard-coded to accept. When the form is submitted, this is what the `params` looks like.

```
{"utf8"=>"✓",
 "authenticity_token"=>"vAsZKoce0jNHjOBWxhHX6QcgesAiEmueBDUgmKmK4gU2nPvnQDTTOdt7bd0oDIO8Libc1ax6QQQ83MhQkvxDVw==",
 "recipe"=>{"name"=>"Pizza"},
 "ingredients"=>[{"name"=>"cheese"},
 {"name"=>"tomatoes"},
 {"name"=>"lettuce"}],
 "commit"=>"Create Recipe"}
```

So, when the user types "Pizza" in the text field for the recipe's name, it's saved in the `params` as `"recipe"=>{"name"=>"Pizza"}`, as shown above. At the same time, when the user enters many values in the text field for the ingredients, it's saved as `"ingredients"=>[{"name"=>"cheese"}, {"name"=>"tomatoes"}, {"name"=>"lettuce"}]` in the `params`, which is an array of hashes.

Where do we go from here? We can use the array of hashes that we get back from `params` and use them to build a recipe with ingredients, thanks to the model association. Since we know what `params` looks like and we only want the value of the recipe hash we can iterate over this hash.

```ruby
class RecipesController < ApplicationController
  def create
    @recipe = Recipe.new(recipe_params)
    params[:recipes].each do |ingredient|
      @recipe.ingredients.build(:name => ingredient[:name])
    end
    if @recipe.save
      # raise params.inspect
      redirect_to recipe_path(@recipe)
    else
      render :new
    end
  end
```

The `@recipe.ingredients.build(:name => ingredient[:name])` is associating each of the ingredients with the corresponding recipe. When the parent object is save the child object is saved automatically too. But we can refactor this a bit more.

Since we are adding an attribute to the ingredient and associating it with a recipe, we can build a custom method in the recipe's model called `ingredients_attributes=(ingredients)`.

```ruby
class Recipe < ActiveRecord::Base
  has_many :pantries
  has_many :ingredients, through: :pantries

  def ingredients_attributes=(ingredients)
    ingredients.each do |ingredient_hash|
      self.ingredients.build(:name => ingredient_hash[:name])
    end
  end
end
```

This custom method takes an argument of hash ingredients and iterates over each hash collecting the name attribute for each ingredient and builds a recipe with each ingredient.

This is how the RecipesController looks like now:

```ruby
class RecipesController < ApplicationController

  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.ingredients_attributes=(params[:ingredients])
    if @recipe.save
      redirect_to recipe_path(@recipe)
    else
      render :new
    end
  end
end
```

Once the submit button is clicked, the rails server output confirms that this action worked.

```
Started POST "/recipes" for ::1 at 2016-10-13 17:26:16 -0400
Processing by RecipesController#create as HTML

  Parameters: {"utf8"=>"✓", "authenticity_token"=>"vAsZKoce0jNHjOBWxhHX6QcgesAiEmueBDUgmKmK4gU2nPvnQDTTOdt7bd0oDIO8Libc1ax6QQQ83MhQkvxDVw==", "recipe"=>{"name"=>"Pizza"}, "ingredients"=>[{"name"=>"cheese"}, {"name"=>"tomatoes"}, {"name"=>"lettuce"}], "commit"=>"Create Recipe"}

   (0.0ms)  begin transaction
  SQL (4.1ms)  INSERT INTO "recipes" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "Pizza"], ["created_at", "2016-10-13 21:26:16.437033"], ["updated_at", "2016-10-13 21:26:16.437033"]]

  SQL (0.0ms)  INSERT INTO "ingredients" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "cheese"], ["created_at", "2016-10-13 21:26:16.441095"], ["updated_at", "2016-10-13 21:26:16.441095"]]

  SQL (0.0ms)  INSERT INTO "pantries" ("recipe_id", "ingredient_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["recipe_id", 7], ["ingredient_id", 7], ["created_at", "2016-10-13 21:26:16.449028"], ["updated_at", "2016-10-13 21:26:16.449028"]]

  SQL (0.0ms)  INSERT INTO "ingredients" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "tomatoes"], ["created_at", "2016-10-13 21:26:16.452992"], ["updated_at", "2016-10-13 21:26:16.452992"]]

  SQL (0.0ms)  INSERT INTO "pantries" ("recipe_id", "ingredient_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["recipe_id", 7], ["ingredient_id", 8], ["created_at", "2016-10-13 21:26:16.456993"], ["updated_at", "2016-10-13 21:26:16.456993"]]

  SQL (0.0ms)  INSERT INTO "ingredients" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "lettuce"], ["created_at", "2016-10-13 21:26:16.461008"], ["updated_at", "2016-10-13 21:26:16.461008"]]

  SQL (0.0ms)  INSERT INTO "pantries" ("recipe_id", "ingredient_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["recipe_id", 7], ["ingredient_id", 9], ["created_at", "2016-10-13 21:26:16.465004"], ["updated_at", "2016-10-13 21:26:16.465004"]]
   (28.7ms)  commit transaction
Redirected to http://localhost:3000/recipes/7
Completed 302 Found in 89ms (ActiveRecord: 32.8ms)
```

We can test this in the rails console to double check.

```
> recipe=Recipe.find(5)
  Recipe Load (0.0ms)  SELECT  "recipes".* FROM "recipes" WHERE "recipes"."id" = ? LIMIT 1  [["id", 5]]
=> #<Recipe id: 5, name: "Pizza", created_at: "2016-10-13 21:22:57", updated_at: "2016-10-13 21:22:57">

> recipe.ingredients
  Ingredient Load (0.0ms)  SELECT "ingredients".* FROM "ingredients" INNER JOIN "pantries" ON "ingredients"."id" = "pantries"."ingredient_id" WHERE "pantries"."recipe_id" = ?  [["recipe_id", 5]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Ingredient id: 1, name: "cheese", created_at: "2016-10-13 21:22:57", updated_at: "2016-10-13 21:22:57">, #<Ingredient id: 2, name: "tomatoes", created_at: "2016-10-13 21:22:57", updated_at: "2016-10-13 21:22:57">, #<Ingredient id: 3, name: "lettuce", created_at: "2016-10-13 21:22:57", updated_at: "2016-10-13 21:22:57">]>
```

Great!

To prevent from saving empty text fields, we can add an `if` statement to the recipe's model `ingredients_attributes=` method:

```ruby
self.ingredients.build(:name => ingredient_hash[:name]) if ingredient_hash[:name].present?
```

Now lets focus on generating the form using rails for the ingredients. So far, this is what we have:

```html
<p>Add Ingredients:</p>
<ul>
  <li><input type="text" name="ingredients[][name]"></li>
  <li><input type="text" name="ingredients[][name]"></li>
  <li><input type="text" name="ingredients[][name]"></li>
</ul>
```

To go one level up of abstraction, we can use a `text_field_tag`.

```html
<p>Add Ingredients:</p>
<ul>
  <li><%= text_field_tag 'ingredients[][name]' %></li>
  <li><%= text_field_tag 'ingredients[][name]' %></li>
  <li><%= text_field_tag 'ingredients[][name]' %></li>
</ul>
```

This will work just the same as the good old html code. But since we're using `ingredients_attributes` in our controller, we can use it to build a form. In this case, a recipe has a hash of ingredients which has an attribute of name and the following will work.

```html
<p>Add Ingredients:</p>
<ul>
  <li><%= text_field_tag 'recipe[ingredients_attributes][][name]' %></li>
  <li><%= text_field_tag 'recipe[ingredients_attributes][][name]' %></li>
  <li><%= text_field_tag 'recipe[ingredients_attributes][][name]' %></li>
</ul>
```

We can mass assign our nested attributes in the controller like this and sanitize these attributes with strong params.

```ruby
class RecipesController < ApplicationController
  def create
    @recipe = Recipe.new(recipe_params)
    @recipe.ingredients_attributes=(params[:recipe][:ingredients_attributes])
    if @recipe.save
      redirect_to recipe_path(@recipe)
    else
      render :new
    end
  end

  private

  def recipe_params
    params.require(:recipe).permit(:name, :ingredients_attributes => [:name])
  end
end
```

Moving one level of abstraction, we can use the `form_for` build to help us create what we're essentially doing, looping through each ingredient in the each text field.

```html
<p>Add Ingredients:</p>
<ul>
  <%= f.fields_for :ingredients do |ingredient_field| %>
    <li><%= ingredient_field.text_field :name %></li>
  <% end %>
</ul>
```

Now we have a new form builder that can build a text field for each ingredient to the recipe. The associated model must respond to a writer method, which we created. If we navigate to the form, it will work but the forms to input the ingredients are missing. To add them, we need to add some code in the RecipesController new action.

```ruby
class RecipesController < ApplicationController
  def new
    @recipe = Recipe.new
    3.times do
      @recipe.ingredients.build
    end
  end
end
```

This is how the html looks like.

```html
<li>
  <input name="recipe[ingredients_attributes][0][name]" id="recipe_ingredients_attributes_0_name" type="text">
</li>
<li>
  <input name="recipe[ingredients_attributes][1][name]" id="recipe_ingredients_attributes_1_name" type="text">
</li>
<li>
  <input name="recipe[ingredients_attributes][2][name]" id="recipe_ingredients_attributes_2_name" type="text">
</li>
```

If you submit the form, you will see this:

```
TypeError in RecipesController#create
no implicit conversion of Symbol into Integer
Rails.root: C:/Users/hmisa/Documents/proj/ror/test/test_nested_attributes
```

This means that we need to allow the index of each ingredients as shown above.

```ruby
def ingredients_attributes=(ingredients)
  ingredients.each do |i, ingredient_hash|
    self.ingredients.build(:name => ingredient_hash[:name])
  end
end
```

And there is our input forms. Now params looks like this.

```
=> {"utf8"=>"✓", "authenticity_token"=>"/wnR...", "recipe"=>{"name"=>"", "ingredients_attributes"=>{"0"=>{"name"=>"cheese"}, "1"=>{"name"=>"tomatoes"}, "2"=>{"name"=>"lettuce"}}}, "commit"=>"Create Recipe", "controller"=>"recipes", "action"=>"create"}
```
