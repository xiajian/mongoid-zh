---
layout: post
title: 查询 
---

Querying

One of MongoDB's greatest features is its ability to execute dynamic queries, which Origin abstracts in a familiar Arel-style DSL that Mongoid includes.

    Queries
    Queries + Persistence
    Scoping
    Find and Modify
    Map/Reduce
    Aggregations
    Geo Near 

Queries

All queries in Mongoid are Criteria, which is a chainable and lazily evaluated wrapper to a MongoDB dynamic query. Criteria only touch the database when they need to, for example on iteration of the results, and when executed wrap a cursor in order to keep memory management and performance predictable.
Queryable DSL

Mongoid's main query DSL is provided by Origin. Any method that is available on an Origin::Queryable exists on a Mongoid::Criteriaas well as off the model's class.

Band.where(name: "Depeche Mode")Band.  where(:founded.gte =< "1980-1-1").  in(name: [ "Tool", "Deftones" ]).  union.  in(name: [ "Melvins" ])

With each chained method on a criteria, a newly cloned criteria is returned with the new query added. This is so that with scoping or exposures, for example, the original queries are unmodified and remain reusable.
	Most of Mongoid's criteria has been extracted into its own gem, Origin, which Mongoid now depends on for most of its API. A complete list of commands can be found in Selection with Origin and Options with Origin. Note that the API has changed from 2.0 - see the Upgrading section for a list of backwards incompatible changes.
Additional Query Methods

In addition to behavior that Origin provides, Mongoid also has some helpful methods on criteria.

Operation
	

Mongoid
	

Moped

Criteria#count

Get a count of persisted documents. Note this will always hit the database for the count.
	

Band.count

Band.where(name: "Photek").count

 
	

collections[:bands].find.count

collections[:bands].find(name: "Photek").count

 

Criteria#distinct

Get a list of distinct values for a single field.
	

Band.distinct(:name)

Band.where(:fans.gt => 100000).

  distinct(:name)

 
	

collections[:bands].find.

  distinct(:name)

collections[:bands].find(fans: { "$gt" => 100000 }).

  distinct(:name)

 

Criteria#each

Iterate over all documents in the collection.
	

Band.each do |band|

  p band.name

end

 
	

collections[:bands].find

 

Criteria#exists?

Determine if any documents exist in the database. Will return true for 1 or more.
	

Band.exists?

Band.where(name: "Photek").exists?

 
	

collections[:bands].find.count

collections[:bands].find(

  { name: "Photek" }

).count

 

Criteria#find

Find a document or multiple documents by their ids. Will raise an error by default if any of the ids do not match.
	

Band.find("4baa56f1230048567300485c")

Band.find(

  "4baa56f1230048567300485c",

  "4baa56f1230048567300485d"

)

Band.where(name: "Photek").find(

  "4baa56f1230048567300485c"

)

 
	

collections[:bands].find(

  { _id: "4baa56f1230048567300485c" }

)

collections[:bands].find(

  { _id:

    { "$in" => [

      "4baa56f1230048567300485c",

      "4baa56f1230048567300485d"

      ]

    }

  }

)

collections[:bands].find(

  {

    _id: "4baa56f1230048567300485c",

    name: "Photek"

  }

)

 

Model.find_by

Find a document by the provided attributes, and if not found raise an error or return nil depending on the raise_not_found_error configuration option.
	

Band.find_by(name: "Photek")

 

Band.find_by(name: "Tool") do |band|

  band.impressions += 1

end

 
	

collections[:bands].find(name: "Photek").first

 

Criteria#find_or_create_by

Find a document by the provided attributes, and if not found create and return a newly persisted one.
	

Band.find_or_create_by(name: "Photek")

Band.where(:likes.gt => 10).find_or_create_by(name: "Photek")

 
	

collections[:bands].find(name: "Photek").first

collections[:bands].find(

  likes: { "$gt" => 10 }, name: "Photek"

).first

 

Criteria#find_or_initialize_by

Find a document by the provided attributes, and if not found initialize and return a new one.
	

Band.find_or_initialize_by(name: "Photek")

Band.where(:likes.gt => 10).find_or_create_by(name: "Photek")

 
	

collections[:bands].find(name: "Photek").first

collections[:bands].find(

  likes: { "$gt" => 10 }, name: "Photek"

).first

 

Criteria#first

Get the first document. If no sort options are provided, Mongoid will add an ascending _id sort to the criteria.
	

Band.first

Band.where(:members.with_size => 3).first

 
	

collections[:bands].find.sort(_id: 1).limit(-1)

collections[:bands].find(

  { members: { "$size" => 3 }}

).sort(_id: 1).limit(-1)

 

Criteria#first_or_create
Since 3.1.0

Find the first document by the provided attributes, and if not found create and return a newly persisted one.
	

Band.where(name: "Photek").first_or_create

 
	

collections[:bands].find(name: "Photek").first

 

Criteria#first_or_create!
Since 3.1.0

Find the first document by the provided attributes, and if not found create and return a newly persisted one using `create!`.
	

Band.where(name: "Photek").first_or_create!

 
	

collections[:bands].find(name: "Photek").first

 

Criteria#first_or_initialize
Since 3.1.0

Find the first document by the provided attributes, and if not found instantiate and return a new one.
	

Band.where(name: "Photek").first_or_initialize

 
	

collections[:bands].find(name: "Photek").first

 

Criteria#for_js

Find documents for a provided javascript expression. This will wrap the javascript in a `Moped::BSON::Code` object will is the safe way to avoid javascript injection attacks.
	

Band.for_js("this.name = param", param: "Tool")

 
	

expr = Moped::BSON::Code.new(

  "this.name = param", param: "Tool"

)

collections[:bands].find("$where" => expr)

 

Criteria#last

Get the last document. If no sort options are provided, Mongoid will add a descending _id sort to the criteria.
	

Band.last

Band.where(:members.with_size => 3).last

 
	

collections[:bands].find.sort(_id: -1).limit(-1)

collections[:bands].find(

  { members: { "$size" => 3 }}

).sort(_id: -1).limit(-1)

 

Criteria#length

Also size. Get a count of persisted documents. After being called once for the criteria, will cache subsequent calls and not hit the database.
	

Band.length

Band.where(name: "Photek").length

 
	

collections[:bands].find.count

collections[:bands].find(name: "Photek").count

 

Criteria#pluck
Since 3.1.0

Get all the non nil values for the provided field.
	

Band.all.pluck(:name)

 

Eager Loading

Mongoid provides a facility to eager load documents from relations to prevent the n+1 issue when iterating over documents with relation access. Eager loaded is supported on all relations with the exception of polymorphic belongs_to associations.

classBand include Mongoid::Document has_many :albumsendclassAlbum include Mongoid::Document belongs_to :bandendBand.includes(:albums).each do |band|  p band.albums.first.name # Does not hit the database again.end

	In order for eager loading to work, the Identity Map must be enabled.
Queries + Persistence

Mongoid supports persistence operations off of criteria in a light capacity for when you want to expressively perform multi document inserts, updates, and deletion.

Operation
	

Mongoid
	

Moped

Criteria#create

Create a newly persisted document.
	

Band.where(name: "Photek").create

 
	

collections[:bands].find(name: "Photek")

collections[:bands].insert(name: "Photek")

 

Criteria#create!

Create a newly persisted document with create!.
	

Band.where(name: "Photek").create!

 
	

collections[:bands].find(name: "Photek")

collections[:bands].insert(name: "Photek")

 

Criteria#build

Create a new document (unsaved). Also new.
	

Band.where(name: "Photek").build

 
	

collections[:bands].find(name: "Photek")

 

Criteria#update

Update attributes of the first matching document.
	

Band.where(name: "Photek").update(label: "Mute")

 
	

collections[:bands].find(

  { name: "Photek" }

).update({ "$set" => { label: "Mute" }})

 

Criteria#update_all

Update attributes of all matching documents.
	

Band.where(name: "Photek").update_all(label: "Mute")

 
	

collections[:bands].find(

  { name: "Photek" }

).update_all({ "$set" => { label: "Mute" }})

 

Criteria#add_to_set

Perform an $addToSet on all matching documents.
	

Band.where(name: "Photek").add_to_set(:label, "Mute")

 
	

collections[:bands].find(

  { name: "Photek" }

).update_all({ "$addToSet" => { label: "Mute" }})

 

Criteria#bit

Perform a $bit on all matching documents.
	

Band.where(name: "Photek").bit(:likes, { and: 14, or: 4 })

 
	

collections[:bands].find(

  { name: "Photek" }

).update_all({ "$bit" => { likes: { and: 14, or: 4} }})

 

Criteria#inc

Perform an $inc on all matching documents.
	

Band.where(name: "Photek").inc(:likes, 123)

 
	

collections[:bands].find(

  { name: "Photek" }

).update_all({ "$inc" => { likes: 123 }})

 

Criteria#pop

Perform a $pop on all matching documents.
	

Band.where(name: "Photek").pop(:members, -1)

Band.where(name: "Photek").pop(:members, 1)

 
	

collections[:bands].find(

  { name: "Photek" }

).update_all({ "$pop" => { members: -1 }})

 

collections[:bands].find(

  { name: "Photek" }

).update_all({ "$pop" => { members: 1 }})

 

Criteria#pull

Perform a $pull on all matching documents.
	

Band.where(name: "Tool").pull(:members, "Maynard")

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all({ "$pull" => { members: "Maynard" }})

 

Criteria#pull_all

Perform a $pullAll on all matching documents.
	

Band.where(name: "Tool").

  pull_all(:members, [ "Maynard", "Danny" ])

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all(

  { "$pullAll" => { members: [ "Maynard", "Danny" ] }}

)

 

Criteria#push

Perform a $push on all matching documents.
	

Band.where(name: "Tool").push(:members, "Maynard")

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all(

  { "$push" => { members: "Maynard" }}

)

 

Criteria#push_all

Perform a $pushAll on all matching documents.
	

Band.where(name: "Tool").

  push_all(:members, [ "Maynard", "Danny" ])

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all(

  { "$pushAll" => { members: [ "Maynard", "Danny" ] }}

)

 

Criteria#rename

Perform a $rename on all matching documents.
	

Band.where(name: "Tool").rename(:name, :title)

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all({ "$rename" => { "name" => "title" }})

 

Criteria#set

Perform a $set on all matching documents.
	

Band.where(name: "Tool").set(:likes, 10000)

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all({ "$set" => { likes: 10000 }})

 

Criteria#unset

Perform a $unset on all matching documents.
	

Band.where(name: "Tool").unset(:likes)

 
	

collections[:bands].find(

  { name: "Tool" }

).update_all({ "$unset" => { likes: true }})

 

Criteria#delete

Delete all matching documents. Note that this will ignore any skip and limit arguments that exist on the criteria.
	

Band.where(label: "Mute").delete

 
	

collections[:bands].find(label: "Mute").remove_all

 

Criteria#destroy

Destroy all matching documents and run callbacks. This will respect skip and limit criteria.
	

Band.where(label: "Mute").destroy

	Be careful with using Criteria#destroy in this context as it will load every matching document into memory in order to run the destroy callbacks.
Scoping

Scopes provide a convenient way to reuse common criteria with more business domain style syntax.
Named Scopes

Named scopes are simply criteria defined at class load that are referenced by a provided name. Just like normal criteria, they are lazy and chainable.

classBand include Mongoid::Document field :country, type: String field :genres, type: Array scope :english, where(country: "England")  scope :rock, where(:genres.in =< [ "rock" ])endBand.english.rock # Get the English rock bands.

Named scopes can take procs and blocks for accepting parameters or extending functionality.

classBand include Mongoid::Document field :name, type: String field :country, type: String field :active, type: Boolean, default: true scope :named, -<(name){ where(name: name) }  scope :active, where(active: true) dodefdeutsch tap |scope| do scope.selector.store("origin" =< "Deutschland") endendendendBand.named("Depeche Mode") # Find Depeche Mode.Band.active.deutsch # Find active German bands.

Default Scopes

Default scopes can be useful when you find yourself applying the same criteria to most queries, and want something to be there by default. Default scopes take either criteria objects or procs that return criteria objects for cases like multi-tenant applications.

classBand include Mongoid::Document field :name, type: String field :active, type: Boolean, default: true default_scope where(active: true)endBand.each do |band| # All bands here are active.end

You can tell Mongoid not to apply the default scope by using unscoped, which can be inline or take a block.

Band.unscoped.where(name: "Depeche Mode")Band.unscoped doBand.where(name: "Depeche Mode")end

You can also tell Mongoid to explicitly apply the default scope again later to always ensure it's there.

Band.unscoped.where(name: "Depeche Mode").scoped

If you are using a default scope on a model that is part of a relation like a has_many, has_and_belongs_to_many, or embeds_many, you must reload the relation to have scoping reapplied. This is important to note if you change a value of a document in the relation that would affect its visibility within the scoped relation.

classLabel include Mongoid::Document embeds_many :bandsendclassBand include Mongoid::Document field :active, default: true embedded_in :label default_scoped where(active: true)endlabel.bands.push(band)label.bands #=< [ band ]band.update_attribute(:active, false)label.bands #=< [ band ] Must reload.label.reload.bands #=< []

Note that you should not use default scopes in conjunction with `Mongoid::Paranoia` as the default scope will override the paranoia scope and cause all documents, soft-deleted or not, to be included.
Class Methods

Class methods on models that return criteria objects are also treated like scopes, and can be chained as well.

classBand include Mongoid::Document field :name, type: String field :active, type: Boolean, default: truedefself.active where(active: true) endendBand.active

Find and Modify

MongoDB's $findAndModify command is a unique atomic "find, perform some operation, and return" operation. In Mongoid, this can be used from the model class level or chained to a criteria.
	The atomic operation that is provided to the find_and_modify operates on the first document it locates in the database. Therefore it is usually important to provide sorting criteria as well.
Execution

Simply call #find_and_modify from the class or the end of a criteria. Unlike other lazy Mongoid operations, this command executes immediately.

Queue.  where(pending: true).  asc(:created_at).  find_and_modify({ "$set" =< { pending: false }}, new: true)

The first parameter to find_and_modify is an atomic update selector, and the optional second parameter is a hash of options. Valid options are:

    new: (true|false) Return the updated document.
    remove: (true|false) Delete the found document.

Map/Reduce

Mongoid provides a DSL around MongoDB's map/reduce framework, for performing custom map/reduce jobs or simple aggregations.
Execution

You can tell Mongoid off the class or a criteria to perform a map/reduce by calling map_reduce and providing map and reduce javascript functions.

map = %Q{ function() { emit(this.name, { likes: this.likes }); }}reduce = %Q{ function(key, values) { var result = { likes: 0 };    values.forEach(function(value) { result.likes += value.likes; });    return result; }}Band.where(:likes.gt =< 100).map_reduce(map, reduce).out(inline: true)

Just like criteria, map/reduce calls are lazily evaluated. So nothing will hit the database until you iterate over the results, or make a call on the wrapper that would need to force a database hit.

Band.map_reduce(map, reduce).out(replace: "mr-results").each do |document|  p document # { "_id" =< "Tool", "value" =< { "likes" =< 200 }}end

The only required thing you provide along with a map/reduce is where to output the results. If you do not provide this an error will be raised. Valid options to #out are:

    inline: 1: Don't store the output in a collection.
    replace: "name": Store in a collection with the provided name, and overwrite any documents that exist in it.
    merge: "name": Store in a collection with the provided name, and merge the results with the existing documents.
    reduce: "name": Store in a collection with the provided name, and reduce all existing results in that collection.

The full API for map/reduce is as follows.
Operation	Mongoid
MapReduce#out

Specify the result output location
	

Band.map_reduce(m, r).out(inline: 1)

MapReduce#counts

Get the count statistics for the map reduce (input, emit, reduce, count).
	

Band.map_reduce(m, r).out(inline: 1).counts

MapReduce#emitted

Get the number of emits.
	

Band.map_reduce(m, r).out(inline: 1).emitted

MapReduce#finalize

Provide a finalize function to run at the end of the job.
	

func = %Q{ function(key, value) { value.extra = true;    return value; }}Band.map_reduce(m, r).out(inline: 1).finalize(func)

MapReduce#input

Get the number of inputs.
	

Band.map_reduce(m, r).out(inline: 1).input

MapReduce#js_mode

Execute the map/reduce in jsMode.
	

Band.map_reduce(m, r).out(inline: 1).js_mode

MapReduce#output

Get the number of outputs.
	

Band.map_reduce(m, r).out(inline: 1).output

MapReduce#reduced

Get the number of reduces.
	

Band.map_reduce(m, r).out(inline: 1).reduced

MapReduce#scope

Provide values to set in the global js scope.
	

Band.map_reduce(m, r).out(inline: 1).scope(field: 10)

MapReduce#time

Returns the execution time.
	

Band.map_reduce(m, r).out(inline: 1).time

Aggregations

Mongoid provides convenience methods for simple aggregations, which can be invoked from the class or criteria. Note that with #min, #max and #sum, if a block is provided it behaves just like they would with a Ruby enumerable.

# Use map/reduce, return a float with the max value.Band.where(:likes.gt =< 100).max(:likes)# Use enumerable, returning the document with the max value.Band.where(:likes.gt =< 100).max do |a,b|  a.likes < > b.likesend

The full API is as follows.
Operation	Mongoid
Criteria#avg

Get the average value for the given field.
	

Band.avg(:likes)Band.where(:likes.gt =< 100).avg(:likes)

Criteria#max

Get the max value for the given field.
	

Band.max(:likes)Band.where(:likes.gt =< 100).max(:likes)

Criteria#min

Get the min value for the given field.
	

Band.min(:likes)Band.where(:likes.gt =< 100).min(:likes)

Criteria#sum

Get the sum of all values for a given field.
	

Band.sum(:likes)Band.where(:likes.gt =< 100).sum(:likes)

Geo Near

Mongoid provides a DSL around MongoDB's $geoNear command.
Execution

You can tell Mongoid off the class or a criteria to perform a $geoNear by calling geo_near and providing an array of [ x, y ] coordinates to search from.

Bar.where(:likes.gt =< 100).geo_near([ 50, 12 ]).spherical

Just like criteria, geo_near calls are lazily evaluated. So nothing will hit the database until you iterate over the results, or make a call on the wrapper that would need to force a database hit. Note that each instantiated document from a $geoNear query will get a special dynamic attribute geo_near_distance that will be available as long as the document is in memory.

Bar.where(:likes.gt =< 100).geo_near([ 50, 12 ]).each do |document|  p document.geo_near_distanceend

The full API for $geoNear is as follows.
Operation	Mongoid	Moped
GeoNear#average_distance

Get the average distance of all the documents from the provided location.
	

Bar.geo_near([ 50, 13 ]).average_distance

	

session.command({  geoNear: "bars",  near: [ 50, 13 ]})["stats"]["averageDistance"]

GeoNear#distance_multiplier

Set the distance multiplier for returned distance calculations.
	

Bar.geo_near([ 50, 13 ]).distance_multiplier(5012)

	

session.command({  geoNear: "bars",  near: [ 50, 13 ],  distanceMultiplier: 5012})

GeoNear#max_distance

Set the maximum distance to return documents for.
	

Bar.geo_near([ 50, 13 ]).max_distance(100)

	

session.command({  geoNear: "bars",  near: [ 50, 13 ],  maxDistance: 100})

GeoNear#spherical

Set the calculations to work for spheres.
	

Bar.geo_near([ 50, 13 ]).spherical

	

session.command({  geoNear: "bars",  near: [ 50, 13 ],  spherical: true})

GeoNear#unique

Specify whether unique documents should be returned.
	

Bar.geo_near([ 50, 13 ]).unique(false)

	

session.command({  geoNear: "bars",  near: [ 50, 13 ],  unique: false})

