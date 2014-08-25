---
layout: post
title: 索引 
---


Indexing

You can define indexes on documents using the index macro. Provide the key for the index along with a direction. For additional options, supply them in a second options hash parameter.

class Person
  include Mongoid::Document
  field :ssn

  index({ ssn: 1 }, { unique: true, name: "ssn_index" })
end

You can define indexes on embedded document fields as well.

class Person
  include Mongoid::Document
  embeds_many :addresses
  index "addresses.street" => 1
end

You can index on multiple fields and provide direction.

class Person
  include Mongoid::Document
  field :first_name
  field :last_name

  index({ first_name: 1, last_name: 1 }, { unique: true })
end

Indexes can be sparse:

class Person
  include Mongoid::Document
  field :ssn

  index({ ssn: -1 }, { sparse: true })
end

Indexes can be run in the background in cases where they may take some time:

class Person
  include Mongoid::Document
  field :ssn
  index({ ssn: 1 }, { unique: true, background: true })
end

For geospatial indexes, make sure the field you are indexing is an Array.

class Person
  include Mongoid::Document
  field :location, type: Array

  index({ location: "2d" }, { min: -200, max: 200 })
end

Indexes can be scoped to a specific database.

class Person
  include Mongoid::Document
  field :ssn
  index({ ssn: 1 }, { database: "users", unique: true, background: true })
end

You can have Mongoid define indexes for you on "foreign key" fields for relational associations. This only works on the relation macro that the foreign key is stored on.

class Comment
  include Mongoid::Document
  belongs_to :post, index: true
  has_and_belongs_to_many :preferences, index: true
end

When you want to create the indexes in the database, use the provided rake task.

rake db:mongoid:create_indexes

Mongoid also provides a rake task to delete all secondary indexes, plus non-defined indexes as well.

rake db:mongoid:remove_indexes


