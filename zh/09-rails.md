---
layout: post
title: rails
---
Rails

Mongoid was built and targeted towards Rails applications, even though it will work in any environment. However if you are using Rails consult the next two sections on how Mongoid hooks into a Rails application.

For a sample Rails application and examples of domain modeling, please see the Mongoid demo application, Echo. Note that currently the application is only models and specs.

    Multi-Parameter Attributes
    Railties
    Rake Tasks 

Multi Parameter Attributes

If you want to use multi-paramater attributes with Rails, you will need to include an extra Mongoid module to support it.
	The reason for needing to include a module is due to the fact that we believe this is not a very usable way of handling date/time entry in forms, with a complex implementation.

classPerson include Mongoid::Document include Mongoid::MultiParameterAttributesend

Railties

Mongoid provides some railties and initializers that one should be aware of when writing a Rails application with Mongoid.
Configuration

You can set Mongoid configuration options in your application.rb along with other Rails environment specific options by accessing config.mongoid. Options set here will override those set in your config/mongoid.yml.

moduleMyApplicationclassApplication <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Rails::Application config.mongoid.logger = Logger.new($stdout, :warn)    config.mongoid.persist_in_safe_mode = trueendend

Model Preloading

In order to properly set up single collection inheritance, Mongoid needs to preload all models before every request in development mode. This can get slow, so if you are not using any inheritance it is recommended you turn this feature off.

config.mongoid.preload_models = false

Exceptions

Similar to Active Record, Mongoid tells raise to return specific http codes when some errors are raised.

    Mongoid::Errors::DocumentNotFound : 404
    Mongoid::Errors::Validations : 422

Unicorn and Passenger

When using Unicorn or Passenger, each time a child process is forked when using app preloading or smart spawning, Mongoid will automatically reconnect to the master database. If you are doing this in your application manually you may remove your code.
Rake Tasks

Mongoid provides the following rake tasks when used in a Rails 3 environment:

    db:create: Exists only for dependency purposes, does not actually do anything.
    db:create_indexes: Reads all index definitions from the models and attempts to create them in the database.
    db:remove_indexes: Reads all secondary index definitions from the models and attempts to remove indexes that are not defined.
    db:drop: Drops all collections in the database with the exception of the system collections.
    db:migrate: Exists only for dependency purposes, does not actually do anything.
    db:purge: Deletes all data, including indexes, from the database. Since 3.1.0
    db:schema:load: Exists only for framework dependency purposes, does not actually do anything.
    db:seed: Seeds the database from db/seeds.rb
    db:setup: Creates indexes and seeds the database.
    db:test:prepare: Exists only for framework dependency purposes, does not actually do anything.



