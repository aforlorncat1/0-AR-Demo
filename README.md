# Use ActiveRecord in your Ruby project

[ActiveRecord](https://github.com/rails/rails/tree/master/activerecord)  is a gem that is part of  [Ruby on Rails](http://rubyonrails.org/). It is the  [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping), i.e. the library that maps our objects to tables. In other words, it is the Ruby library that allows us to use Ruby classes in order to access our data stored in an RDBMS, like SQLite, MySQL or PostgreSQL.


## Initiate a Ruby Project

Let's start with initialization of our Ruby project. We are going to create a Ruby application that manages movies. Hence, let's call the project  `movies_app`.

Create the root folder of the project. Let's call it movies_app

~~~
$ mkdir movies_app
$ cd movies_app
movies_app $
~~~

## Create  `Gemfile`

We will now create the  `Gemfile`  and add the activerecord gem in. Also, we will add the gem  [standalone_migrations](https://github.com/thuss/standalone-migrations). This gem is very useful, because it allows us to execute  `rake`  tasks related to migrations.

# Gemfile
~~~
source '[https://rubygems.org](https://rubygems.org/)'

gem 'activerecord'
gem 'standalone_migrations'
~~~

## Install Gems

Having created the  `Gemfile`, now let's proceed to the installation of the necessary gems. Run  `bundle`and you will see the gems downloaded and installed.

~~~
movies_app $ bundle
Fetching gem metadata from [https://rubygems.org/](https://rubygems.org/)...............
Resolving dependencies...
Fetching rake 12.3.0
...
Fetching standalone_migrations 5.2.3
Installing standalone_migrations 5.2.3
Bundle complete! 2 Gemfile dependencies, 27 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
movies_app $
~~~

## Decide On Your Database

Now, you will need to choose your database. Is it going to be SQLite3? Is it going to be PostgreSQL? For now, let's use SQLite3. This means that I need one more gem in my Gemfile. The  [sqlite3](https://rubygems.org/gems/sqlite3)  gem.

This is the new version of my  `Gemfile`:

# Gemfile
source '[https://rubygems.org](https://rubygems.org/)'

~~~
gem 'activerecord'
gem 'standalone_migrations'
gem 'sqlite3'

And then I do a  `bundle`  again.

movies_app $ bundle
...
Fetching sqlite3 0.21.0
Installing sqlite3 0.21.0 with native extensions
Using thor 0.20.0
Using railties 5.1.4
...
movies_app $
~~~

## Create Database Configuration File

With the gem for the database access in place, now lets create the database configuration file. This is going to be a file inside the folder  `db`  and having the name  `config.yml`. Hence, create the file  `db/config.yml`  with the following content:

~~~
default: &default
  adapter: sqlite3
  encoding: unicode
  pool: 5

development:
  <<: *default
  database: db/movies_development.sqlite3

test: &test
  <<: *default
  database: db/movies_test.sqlite3

production:
  <<: *default
  database: db/movies_production.sqlite3
~~~

The above  `db/config.yml`  file is an indicative file for the sqlite3 database. You should adapt the content of this file according to your database server.

## Create Rakefile

Now go ahead and create a  `Rakefile`  at the root folder of your project. This file needs to have the following content:

# Rakefile
~~~
require 'standalone_migrations'
StandaloneMigrations::Tasks.load_tasks
~~~

It will load all the tasks that will help you create and manage your migrations.

To double check that you have access to the tasks, run the following command:

~~~
movies_app $ bundle exec rake --tasks
~~~

You will get a long list of the tasks that you have available.

## Create Database

The next step is that you create your local database:

~~~
movies_app $ bundle exec rake db:create
Created database 'movies_development'
Created database 'movies_test'
movies_app $
~~~

The above rake task created both a development and a test database

## Create First Migration

Now, let's create the first db migration. We are going to use a rake task:

~~~
movies_app $ bundle exec rake db:new_migration[create_movies]
      create  db/migrate/20190702051113_create_movies.rb
movies_app $      
~~~

## Edit Migration And Invoke

Let's edit the file  `db/migrate/20190702051113_create_movies.rb`

~~~
class CreateMovies < ActiveRecord::Migration[5.1]
  def change
    create_table :movies do |t|
      t.string :title, null: false
      t.string :director, null: false

      t.timestamps
    end
  end
end
~~~

The above is a very simple db schema migration. It creates the table  `movies`  with two main columns and the columns for the timestamps.

Let's invoke the migration:

~~~
movies_app $ bundle exec rake db:migrate
(in /Users/recode/Documents/movies_app)
== 20190702051113 CreateMovies: migrating =====================================
-- create_table(:movies)
   -> 0.0063s
== 20190702051113 CreateMovies: migrated (0.0064s) ============================
             
movies_app $
~~~

Nice! The migration has been executed successfully.

## Create a Model

Let's see now how we can create an ActiveRecord model that would allow us to manage movies. Create the file  `app/models/movie.rb`  with the following content:

### app/models/movie.rb
~~~
class Movie < ActiveRecord::Base
  validates :title, presence: true, uniqueness: {case_insensitive: true}
  validates :director, presence: true
end
~~~
It is very simple. A class that derives from  `ActiveRecord::Base`.

## Our Main Application

Finally, let's create a main file that would use the  `Movie`  model to create a movie in the database. Create the file  `app/main.rb`  with the following content:

~~~
require 'active_record'
require_relative './models/movie'

def db_configuration
  db_configuration_file = File.join(File.expand_path('..', __FILE__), '..', 'db', 'config.yml')
  YAML.load(File.read(db_configuration_file))
end

ActiveRecord::Base.establish_connection(db_configuration["development"])

print "Give me the title of the movie: "
title = gets.chomp

print "Give me the director of the movie: "
director = gets.chomp

title = Movie.new(title: title, director: director)
title.save!
 
puts "Number of movies in your database: #{Movie.count}"
puts "Bye!"
~~~

This is a very simple program that uses ActiveRecord to connect to your database and then create a Movie based on the data provided by the user. The lines with the configuration and establishing the connection are necessary in order for you to establish a database connection. It loads the database connection configuration data from the  `db/config.yml`  file and then picks up the  `development`  environment part and sends it to  `ActiveRecord::Base.establish_connection`  method. Needless to say, that this code needs to be adapted to read the environment part dynamically.

An instance of running this program is given below:

~~~
movies_app $ bundle exec ruby app/main.rb 
Give me the title of the movie: Jurassic Park
Give me the director of the movie: Steven Spielberg
Number of movies in your database: 1
Bye!
movies_app $
~~~

# Closing Note

The above is the bare minimum that would allow you to incorporate ActiveRecord into your Ruby (non-Rails) application. The help of the gem  `standalone_migrations`  is very valuable because it allows us to use the ActiveRecord Migrations API and allows us access to convenient Rake tasks.