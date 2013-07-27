---
layout: post
title: "Dealing with Associations in Rails 3"
date: 2013-07-27 00:17
comments: true
categories: [hacks]
---

At some point in constructing your Rails application, you'll need to deal with
the various types of associations among your objects. Some objects may belong to others, or
some objects may possess many other objects. You have to handle associations
among your objects in ways that make sense by developing your resources, routes,
models, and database tables around their various relationships.

This guide goes over the types of relationships that exist among objects and
introduces the new Rails developer to ways in which you can define these associations
in various aspects of your Rails models, routes, database, and controllers.



* [Intro to Associations](#associations)
  - [One-to-One](#associations-onetoone)
  - [One-to-Many](#associations-onetomany)
  - [Many-to-Many](#associations-manytomany)
  - [Aside: Polymorphic Associations](#polymorphic)
* [Define Associations in your Models](#models)
  - [One-to-One](#models-onetoone)
  - [One-to-Many](#models-onetomany)
  - [Many-to-Many](#models-manytomany)
* [Configure Nested Resources](#nestedresources)
* [Configure your Database with Foreign Keys](#databaseconfig)
* [Utilize Association Methods](#associationmethods)




  ##Intro to Associations<a id="associations"></a>

	Two different objects, A and B, can have one of several types of relationships with one another:

	* **one-to-one**<a id="associations-onetoone"></a>: Object A has exactly one of object B, while object B has exactly one of object A.
For example, every employee has a single resume, while every resume has a single employee;
every left shoe has a right shoe, and every right shoe has a left shoe, etc.
Such relationships are *bijective*: you would not have a right shoe
without a left shoe, *and* you would not have a left shoe without a right shoe.

	* **one-to-many**<a id="associations-onetomany"></a>: Object A has many of object B, whereas a single object B
would belong to some object A. For example, a company has many employees, and
every employee belongs to some company. There is a one-to-many relationship
between libraries and books; solar systems and planets;
mothers and daughters; and Lady Gaga and Lady Gaga fans. One-to-many relationships
are *subjective*: for every employee, there is always a company that the employee
belongs to, but it is not (always) the case that a given company contains a
given employee. Every book belongs on shelf, but not always on any particular shelf
that you choose.

	* **many-to-many**<a id="associations-manytomany"></a>: An object A can be associated with multiple object Bs,
while an object B can be associated with multiple object As.
An author can write several books, while a book can have several authors.
A class can have many students, while a student can have many classes.
Over the course of time, a person can borrow many library books, while a book 
can be lent to many people.

	**Aside: Polymorphic Associations**<a id="polymorphic"></a>

	Say you are designing Facebook and you want a Like to be able to belong to
either a status, photo, or video.

	Instead of creating separate models of likes to define likes
which belong to a status, likes which belong to a photo, and likes which
belong to a video, you can define a **polymorphic association** between
likes and status, photos & videos which are a many-to-one relatonship
between likes and all statuses/photos/videos. Polymorphic associations
allow you to create just one Like class without defining multiple
Like models which essentially have the same attributes and behaviors, allowing
you to define relationships among objects that transcend class types.

	A status, photo, or video has many likes, but they are all likeable.
You can define a polymoprhic association using the `:polymorphic` option
of `belongs_to`. For example,


			class Like < ActiveRecord::Base
				belongs_to :likable, :polymoprhic => true
				belongs_to :user
			end


	See more about defining polymorphic associations in a future post.


  ##Define Associations in your Models<a id="models"></a>


	In defining relationships among classes in your model, Rails uses
the following helper methods for each given
association type (note the use of plural and singular forms):

	 - **In one-to-one**<a id="models-onetoone"></a>


				class Employee < ActiveRecord::Base
					has_one :resume
				end
		  		   
				class Resume < ActiveRecord::Base
					has_one :employee
				end


	 - **In one-to-many**<a id="models-onetomany"></a>


				class Magazine < ActiveRecord::Base
					has_many :ads
				end
    
				class Ad < ActiveRecord::Base
					belongs_to :magazine
				end      
    
   	 
	 - **In many-to-many**<a id="models-manytomany"></a>


				class Books < ActiveRecord::Base
		  		has_many :borrowers
				end
    
				class Borrower < ActiveRecord::Base
		  		has_many :books
				end      
    

  ##Configure Nested Resources<a id="nestedresources"></a>


	You may want to nest your resources when dealing with models in a 
many/one-to-many relationship in order to define routes
like *like www.example.com/magazines/ads*. 


	In config/routes.rb,


		resources :magazines do
			resources :ads
		end



  ##Configure your Database with Foreign Keys<a id="databaseconfig"></a>

	Define the associations between a magazine and an ad in the database by creating
a column of foreign keys in the ads table called `magazine_id`.  Run a migration
like the following:


		class AddMagazineIdToAds < ActiveRecord::Migration
			def change
				add_column :ads, :magazine, :references
				add_index :ads, :magazine_id
			end
		end


	The `references` part is a shortcut that adds an integer column acting as
	your foreign key. It is called `magazine_id` in this case. ActiveRecord
	determines the title of your foreign key by taking the string form of the
	argument, `:magazine` and adding a suffix `_id`. You equally could have written:


		class AddMagazineIdToAds < ActiveRecord::Migration
			def change
				add_column :ads, :magazine_id, :integer
				add_index :ads, :magazine_id
			end
		end

 

	Indexing `magazine_id` allows our database to search through `magazine_id` more quickly.

	When you call `@magazine.ads`, ActiveRecord searches through the ads in your
ads table which contain the `magazine_id` matching your primary key `id`
belonging to `@magazine` in your `magazines` table. 

	If you use SQL as your database, then if the `id` or primary key for `@magazine`
	in your magazines table is equal to 1, calling `@magazine.ads` would run 
	the SQL query
	

		SELECT * FROM ads WHERE magazine_id = 1
    
    
	to collect all ads belonging to `@magazine`.


  ##Utilize Association Methods<a id="associationmethods"></a>

	Defining associations in your models automatically gives you access to a slew
	of useful association methods which will be 
	particularly helpful as you develop your controller actions.
	
	If you have a one-to-many relationship like	
	
		class Magazine < ActiveRecord::Base
			has_many :ads
		end
    
		class Ad < ActiveRecord::Base
			belongs_to :magazine
		end      

	`has_many` creates an `ads` method on any given Magazine object.
	`@magazine.ads` returns a collection of `ads` that belong to `@magazine`.
	`has_many` also gives you useful helper methods, such as `@ad = @magazine.ads.build`
which instantiates a new record `@ad` that associates itself with `@magazine`,
allowing you to add an ad to `@magazines.ads` without having to call `@ad.save`.

	There are many other useful association methods you can learn 
about on the [Ruby on Rails API](http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html).


##Learn More


Rails provides many useful built-in features for managing associations.
This brief introduction walks through a few of the key
concepts and know-how of implementing associations in Rails.
[Railscast](railscast.com) offers excellent video tutorials in [Nested Resources](http://railscasts.com/episodes/139-nested-resources),
[Nested Models](http://railscasts.com/episodes/196-nested-model-form-revised),
and [Polymorphic Associations](http://railscasts.com/episodes/154-polymorphic-association)
for a better understanding of how associations work in Rails.
Hopefully by now, you can get started on creating assocations in your Rails app.
Best of luck.


