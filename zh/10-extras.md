---
layout: post
title: extras
---


Extras

Mongoid has some useful extra features that can be used in applications.

    Caching
    Paranoia
    Versioning
    Timestamping

Caching

Out of the box, Mongoid wraps the MongoDB Ruby Driver's cursor in order to be memory efficient for large queries and data sets. However if you want the query to load all matching documents in memory and return them on n iterations without hitting the database again you may cache a criteria:

To cache on a per-query basis:

Person.where(first_name: "Franziska").cache

Paranoid Documents Removing in 4.0.0

There may be times when you don't want documents to actually get deleted from the database, but "flagged" as deleted. Mongoid provides a Paranoia module to give you just that.

class Person
  include Mongoid::Document
  include Mongoid::Paranoia
end

person.delete   # Sets the deleted_at field to the current time, ignoring callbacks.
person.delete!  # Permanently deletes the document, ignoring callbacks.
person.destroy  # Sets the deleted_at field to the current time, firing callbacks.
person.destroy! # Permanently deletes the document, firing callbacks.
person.restore  # Brings the "deleted" document back to life.

The documents that have been "flagged" as deleted (soft deleted) can be accessed at any time by calling the deleted class method on the class.

Person.deleted # Returns documents that have been "flagged" as deleted.

Versioning Removing in 4.0.0

Mongoid supports simple versioning through inclusion of the Mongoid::Versioning module. Including this module will create a versions embedded relation on the document that it will append to on each save. It will also update the version number on the document, which is an integer.

class Person
  include Mongoid::Document
  include Mongoid::Versioning
end

You can also set a max_versions setting, and Mongoid will only keep the max most recent versions.

class Person
  include Mongoid::Document
  include Mongoid::Versioning

  # keep at most 5 versions of a record
  max_versions 5
end

You may skip versioning at any point in time by wrapping the persistence call in a versionless block.

person.versionless do |doc|
  doc.update_attributes(name: "Theodore")
end

Timestamping

Mongoid supplies a timestamping module in Mongoid::Timestamps which can be included to get basic behavior for created_at and updated_at fields.

class Person
  include Mongoid::Document
  include Mongoid::Timestamps
end

You may also choose to only have specific timestamps for creation or modification.

class Person
  include Mongoid::Document
  include Mongoid::Timestamps::Created
end

class Post
  include Mongoid::Document
  include Mongoid::Timestamps::Updated
end

If you want to turn off timestamping for specific calls, use the timeless method:

person.timeless.save
Person.timeless.create!

If you'd like shorter timestamp fields with aliases on them to save space, you can include the short versions of the modules.

class Band
  include Mongoid::Document
  include Mongoid::Timestamps::Short # For c_at and u_at.
end

class Band
  include Mongoid::Document
  include Mongoid::Timestamps::Created::Short # For c_at only.
end

class Band
  include Mongoid::Document
  include Mongoid::Timestamps::Updated::Short # For u_at only.
end


