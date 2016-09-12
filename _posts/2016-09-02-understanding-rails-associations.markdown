---
layout: post
title:  "Understanding Rails Associations"
date:   2016-09-02
categories: Rails
---

## How I Understand Associations in Rails

To understand the way associations work in `rails`, I had to use a domain model that I understood as an example. In this case, a recipe that is made up of ingredients. So, here is how I broke it down to cement this concept.

As I thought about how to make the tables in the database to relate to one another, I ran into a problem. I needed a way to express that a recipe can have lots of ingredients. So, the easiest way to do this was to create a column named `recipe_id` in the ingredients_table, referencing which recipe this ingredient belongs to. This would look like this:

ingredients_table
| id  | name     | recipe_id |
| :-- | :------- | :---------|
| 1   | Tomatoes | 1         |

recipes_table
| id  | name     |
| :-- | :------- |
| 1   | Lasagna  |

But what would happen when I want to save my grandma's Pot Pie recipe, that also uses tomatoes, in the database?

ingredients_table
| id  | name     | recipe_id |
| :-- | :------- | :---------|
| 1   | Tomatoes | 1, [2]    |

recipes_table
| id  | name     |
| :-- | :------- |
| 1   | Lasagna  |
| 2   | Pot Pie  |

Now, I'm forced to create another column in the ingredients_table to hold the ingredient from my grandma's recipe or spell tomatoes differently, maybe by dropping the "e". But thats very sloppy and a nightmare to maintain. It seems that I have a many to many relationship among these two tables, recipes_table and ingredients_table. So the best solution, in this case, is to create a `join table`, which I called `pantries`.

Here is how the tables looks now:

ingredients_table
| id  | name     |
| :-- | :------- |
| 1   | Tomatoes |

pantries_table
| id  | recipe_id | ingredient_id |
| :-- | :-------- | :-------------|
| 1   | 1         | 1             |
| 2   | 2         | 1             |

recipes_table
| id  | name     |
| :-- | :------- |
| 1   | Lasagna  |
| 2   | Pot Pie  |

Now I can add my grandma's Pot Pie recipe, that needs those tomatoes to taste great, to the database. But this time, the ingredient that is associated to the Pot Pie recipe, will be stored the `join table` called pantries. As you can see, the Lasagna and the Pot Pie use `tomatoes` in their recipes. And now, both recipes are pointing to the same ingredient in the ingredients_table, all possible through the `join table` foreign key columns.

### Putting This Understanding Into Practice

To connect all these tables together so `rails` can talk to the database, I need to include the macros in each corresponding model as shown below.

### Ingredient Model

```ruby
class Ingredient < ActiveRecord::Base
  has_many :pantries
  has_many :recipes, through: :pantries
end
```

To create an ingredient:

```
$ ingredient = Ingredient.create name: "Carrots"

(0.2ms)  SAVEPOINT active_record_1
SQL (0.3ms)  INSERT INTO "ingredients" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "Carrots"], ["created_at", "2016-09-12 03:00:54.466500"], ["updated_at", "2016-09-12 03:00:54.466500"]]
(0.1ms)  RELEASE SAVEPOINT active_record_1
=> #<Ingredient id: 2, name: "Carrots", created_at: "2016-09-12 03:00:54", updated_at: "2016-09-12 03:00:54">
```

To create an ingredient with its associated recipe:

```
$ ingredients.recipes.create name: "Pot Pie"

(0.2ms)  SAVEPOINT active_record_1
SQL (0.2ms)  INSERT INTO "recipes" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "Pot Pie"], ["created_at", "2016-09-12 03:02:59.150882"], ["updated_at", "2016-09-12 03:02:59.150882"]]
SQL (0.3ms)  INSERT INTO "pantries" ("ingredient_id", "recipe_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["ingredient_id", 2], ["recipe_id", 2], ["created_at", "2016-09-12 03:02:59.154880"], ["updated_at", "2016-09-12 03:02:59.154880"]]
(0.1ms)  RELEASE SAVEPOINT active_record_1
=> #<Recipe id: 2, name: "Pot Pie", created_at: "2016-09-12 03:02:59", updated_at: "2016-09-12 03:02:59">
```

To check the ingredient's recipes:

```
$ ingredient.recipes

Recipe Load (0.5ms)  SELECT "recipes".* FROM "recipes" INNER JOIN "pantries" ON "recipes"."id" = "pantries"."recipe_id" WHERE "pantries"."ingredient_id" = ?  [["ingredient_id", 2]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Recipe id: 2, name: "Pot Pie", created_at: "2016-09-12 03:02:59", updated_at: "2016-09-12 03:02:59">]>
```

### Pantry Model

```ruby
class Pantry < ActiveRecord::Base
  belongs_to :recipe
  belongs_to :ingredient
end
```

To check the Pantry's associations:

```
 $ Pantry.all

 Pantry Load (0.3ms)  SELECT "pantries".* FROM "pantries"
=> #<ActiveRecord::Relation [#<Pantry id: 1, recipe_id: 1, ingredient_id: 1, created_at: "2016-09-12 02:40:19", updated_at: "2016-09-12 02:40:19">, #<Pantry id: 2, recipe_id: 2, ingredient_id: 2, created_at: "2016-09-12 03:02:59", updated_at: "2016-09-12 03:02:59">]>
```

If you want to create a Pantry then assign a associated recipe and ingredient

```
$ pantry = Pantry.create

(0.1ms)  SAVEPOINT active_record_1
SQL (0.3ms)  INSERT INTO "pantries" ("created_at", "updated_at") VALUES (?, ?)  [["created_at", "2016-09-12 03:09:26.118106"], ["updated_at", "2016-09-12 03:09:26.118106"]]
(0.2ms)  RELEASE SAVEPOINT active_record_1
=> #<Pantry id: 3, recipe_id: nil, ingredient_id: nil, created_at: "2016-09-12 03:09:26", updated_at: "2016-09-12 03:09:26">
```

What the console outputs: Here an SQL statement is been executed. It says, "INSERT this timestamp value of "2016-09-12 00:41:27.424544" INTO the "created_at" column of the "pantries" table". The same goes with the "updated_at" column.

```
$ pantry.recipe = pizza

=> #<Recipe id: 3, name: "Pizza", created_at: "2016-09-12 03:11:20", updated_at: "2016-09-12 03:11:20">

$ pantry

=> #<Pantry id: 3, recipe_id: 3, ingredient_id: nil, created_at: "2016-09-12 03:09:26", updated_at: "2016-09-12 03:09:26">

$ pantry.ingredient = cheese

=> #<Ingredient id: 3, name: "Cheese", created_at: "2016-09-12 03:11:57", updated_at: "2016-09-12 03:11:57">

$ pantry

=> #<Pantry id: 3, recipe_id: 3, ingredient_id: 3, created_at: "2016-09-12 03:09:26", updated_at: "2016-09-12 03:09:26">
```

### Recipe Model

```ruby
class Recipe < ActiveRecord::Base
  has_many :pantries
  has_many :ingredients, through: :pantries
end
```

To create a recipe:

```
$ recipe = Recipe.create name: "Lasagna"

(0.1ms)  SAVEPOINT active_record_1
SQL (0.5ms)  INSERT INTO "recipes" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "Lasagna"], ["created_at", "2016-09-12 02:36:47.743490"], ["updated_at", "2016-09-12 02:36:47.743490"]]
(0.1ms)  RELEASE SAVEPOINT active_record_1
=> #<Recipe id: 1, name: "Lasagna", created_at: "2016-09-12 02:36:47", updated_at: "2016-09-12 02:36:47">
```

To create a recipe with its associated ingredient:

```
$ recipe.ingredients.create name: "Tomatoes"

(0.1ms)  SAVEPOINT active_record_1
SQL (0.4ms)  INSERT INTO "ingredients" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "Tomatoes"], ["created_at", "2016-09-12 02:40:19.153429"], ["updated_at", "2016-09-12 02:40:19.153429"]]
SQL (0.3ms)  INSERT INTO "pantries" ("recipe_id", "ingredient_id", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["recipe_id", 1], ["ingredient_id", 1], ["created_at", "2016-09-12 02:40:19.167678"], ["updated_at", "2016-09-12 02:40:19.167678"]]
(0.1ms)  RELEASE SAVEPOINT active_record_1
=> #<Ingredient id: 1, name: "Tomatoes", created_at: "2016-09-12 02:40:19", updated_at: "2016-09-12 02:40:19">
```

To check the recipe's ingredients:

```
$ recipe.ingredients

Ingredient Load (0.4ms)  SELECT "ingredients".* FROM "ingredients" INNER JOIN "pantries" ON "ingredients"."id" = "pantries"."ingredient_id" WHERE "pantries"."recipe_id" = ?  [["recipe_id", 1]]
=> #<ActiveRecord::Associations::CollectionProxy [#<Ingredient id: 1, name: "Tomatoes", created_at: "2016-09-12 02:40:19", updated_at: "2016-09-12 02:40:19">]>
```

#######   #####    #######

Here is the schema of my recipe_test models:

```ruby
ActiveRecord::Schema.define(version: 20160911232947) do

  create_table "ingredients", force: :cascade do |t|
    t.string   "name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

  create_table "pantries", force: :cascade do |t|
    t.integer  "recipe_id"
    t.integer  "ingredient_id"
    t.datetime "created_at",    null: false
    t.datetime "updated_at",    null: false
  end

  create_table "recipes", force: :cascade do |t|
    t.string   "name"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
  end

end
```
