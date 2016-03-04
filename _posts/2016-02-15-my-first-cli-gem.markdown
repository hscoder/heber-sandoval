---
layout: post
title:  "My first CLI Gem"
date:   2016-02-15
categories: CLI Gem
---
For my first CLI Gem project, I decided to scrape a website that contains lots of recipes. Then, using a simple CLI application, users can interact with different recipes and get more information, if they desire

In order to do this, I needed to get some gems that would make this easier. So, from my terminal, I typed `gem install bundler` This allowed me to create the structure for my project using the following command: `bundle gem recipe`. After all files and directories were created, it was time to configure some of the logic in the files.

The first file, `recipe`, is located in the bin directory, which is how the user will interact with the program.

`Recipe::CLI.new.call`

To run the program, the user types `./bin/recipe`.

Before I can run the program, I created these files to collaborate with each other under the `lib` directory.

`cli.rb   my_recipe.rb   scraper.rb`

The `my_recipe` class is responsible for creating its own objects with contains a name, description and url property. Also, it saves itself into the `@@all` class variable, which is set to an empty array.

The `scraper` class is responsible for scraping the needed data from a website. Once this class grabs the desired information, using Nokogiri, its methods will separate each piece of information into a format that the user can read.

The `get_page` method will grab the entire HTML document passed in as an argument.

{% highlight ruby %}
def get_page
  doc = Nokogiri::HTML(open("http://www.example.com"))
end
{% endhighlight %}

In the next method, `get_recipes`, a css selector is being used (via the `.css` method) to grab all of the HTML elements that contain a recipe name.

{% highlight ruby %}
def get_recipes
  self.get_page.css("li")
end
{% endhighlight %}

Now that the stage is set, it's time to make some recipes! Well, not literally, unless if you're a chef. The `make_recipes` takes care of that. First, it iterates through the elements that `get_recipes` returns and then instantiating a `MyRecipe` instance assigning its respective value to the object's attributes.

{% highlight ruby %}
def make_recipes
  get_recipes.each do |rec|
    if rec.css("i").text != ""
      recipe = Recipe::MyRecipe.new
      recipe.name = rec.css("a").text
      recipe.description = rec.css("i").text
      recipe.url = rec.css("a").attr("href").value
    end
  end
end
{% endhighlight %}

Where can we implement the logic for how the user will request recipe information? This will happen in the `cli`class. When the program is run, via the `./bin/recipe`, the `Recipe::CLI.new.call` is invoked. What's happening here? An instance is being create and it calls the `call` method. But what is this method doing? Well, I hate to say this, but the user is not allow know. So we'll keep that implementation encapsulated. Just like in a restaurant, you go to eat, not to mingle with the chef. Can you imagine what would happen if we're allowed in the kitchen? That's why we have waiters. You order something, the waiter goes to the kitchen with your request and minutes later, you get your food. But for the purpose of this blog, let's take a peek.

Here the `call` method will invoke four methods, as shown below:

{% highlight ruby %}
def call
    scrape_recipes
    list_recipes
    menu
    goodbye
  end
{% endhighlight %}

The first method it will call is the `scrape_recipes` method. Here, a instance of the Scraper class is being instantiated.

{% highlight ruby %}
  def scrape_recipes
    new_scaprer = Recipe::Scraper.new
    new_scaprer.get_page
    new_scaprer.get_recipes
    new_scaprer.make_recipes
  end
{% endhighlight %}

When the user types `"list"` in the terminal, it will output:

{% highlight ruby %}
#=> '1. Beef Stroganoff' to STDOUT.
{% endhighlight %}

See below:

{% highlight ruby %}
  def list_recipes
    @recipes = Recipe::MyRecipe.all
    @recipes.each.with_index(1) do |recipe, i|
      puts "#{i}. #{recipe.name}"
    end
  end
{% endhighlight %}

The `menu` method will take user input and check for some basic validation. If the input is valid, then the code associated with the statement will be run. If the user types `exit`, the `goodbye` method will get called.

{% highlight ruby %}
  def menu
    input = nil
    while input != "exit"
      puts "Enter a recipe number for more information or type 'list' to see the recipes or type 'exit':"
      input = gets.strip.downcase

      if input.to_i > 0
        the_recipe = @recipes[input.to_i - 1]
        puts "#{the_recipe.name}"
        puts "#{the_recipe.description}"
        puts "#{the_recipe.url}"
      elsif input == "list"
        list_recipes
      else
        puts "Not sure what you want, type 'list' or 'exit'."
      end

    end
  end
{% endhighlight %}

{% highlight ruby %}
  def goodbye
    puts "See you tomorrow with more exciting recipes"
  end
{% endhighlight %}

That explains, more or less, how this program works. Thought this gem is very simple, I welcome any improvements, suggestions or tips my way.
