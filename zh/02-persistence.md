---
layout: post
title:  持久化(Persistence)
---

Mongoid supports all expected CRUD operations for those familiar with other Ruby mappers like Active Record or Data Mapper. What distinguishes Mongoid from other mappers for MongoDB is that the general persistence operations perform atomic updates on only the fields that have changed instead of writing the entire document to the database each time.

Mongoid支持所有期待的CURD的操作。

The persistence sections will provide examples on what database operation is performed when executing the documented command.

    Standard
    Atomic
    Custom 

## Standard
----

Mongoid's standard persistence methods come in the form of common methods you would find in other mapping frameworks. The following table is a cheat sheet with the method in Mongoid on the left, and the Moped driver operation on the right.

------------------------------------- | ------------------------------------------------------------------------------- 
![](/mongoid-zh/images/achtung.png)   |	Mongoid never persists the entire document at once unless it is new. It figures out what has changed, and only ever updates the changed items atomically.


# 关于如何使用复杂的表格，即表格中插代码的方式，并没有找到合适的方式，还是原先提供的文档的格式比较好。

 Operation            |          Mongoid                    |                         Moped
--------------------- | ----------------------------------- | --------------------------------------------
`Model.create`<br/>Insert a document or multiple documents into the database	 |{% highlight ruby %}
Person.create(
  first_name: "Heinrich",
  last_name: "Heine"
)

Person.create([
  { first_name: "Heinrich", last_name: "Heine" },
  { first_name: "Willy", last_name: "Brandt" }
])

Person.create(first_name: "Heinrich") do |doc|
  doc.last_name = "Heine"
end {% endhighlight %} |{% highlight ruby %} 
collections[:people].insert({
  first_name: "Heinrich",
  last_name: "Heine"
})
collections[:people].insert([
  { first_name: "Heinrich", last_name: "Heine" },
  { first_name: "Willy", last_name: "Brandt" }
]){% endhighlight %}
 

## Atomic Persistence
----

Although Mongoid performs atomic operations under the covers by default, there may be cases where you want to do this explicitly without persisting other fields. Mongoid provides support for all of these operations as well.

------------------------------------- | ------------------------------------------------------------------------------- 
![](/mongoid-zh/images/achtung.png)   | When executing atomic operations via these methods, no callbacks will ever get run, nor will any validations.


   Operation                                                      | 	    Mongoid                        | 	Moped  
----------------------------------------------------------------- | ------------------------------------ | ------------------------------------------
`Model#add_to_set`<br/>Performs an atomic $addToSet on the field. | person.add_to_set(:aliases, "Bond")  | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$addToSet" => { aliases: "Bond" })
`Model#bit`<br/>Performs an atomic $bit on the field.             | person.bit(:age, { and: 10, or: 12 }) |  collections[:people].find(...).<br/> &nbsp;&nbsp;update("$bit" => { age: { and: 10, or: 12 }})
`Model#inc`<br/>Performs an atomic $inc on the field.             | person.inc(age: 1)                   | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$inc" => { age: 1 })
`Model#pop`<br/>Performs an atomic $pop on the field.             | person.pop(aliases: 1)               | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$pop" => { aliases: 1 })
`Model#pull`<br/>Performs an atomic $pull on the field.           | person.pull(aliases: "Bond")         | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$pull" => { aliases: "Bond" })
`Model#pull_all`<br/>Performs an atomic $pullAll on the field.    | person.pull_all(:aliases, [ "Bond", "James" ]) | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$pullAll" => { aliases: [ "Bond", "James" ]})
`Model#push`<br/>Performs an atomic $push on the field.           | person.push(aliases: "007") | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$push" => { aliases: "007" })
`Model#push_all`<br/>Performs an atomic $pushAll on the field.    | person.push_all(:aliases, [ "007", "008" ]) | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$pushAll" => { aliases: [ "007", "008" ]})
`Model#rename`<br/>Performs an atomic $rename on the field.       | person.rename(bday: :dob) | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$rename" => { "bday" => "dob" })
`Model#set`<br/>Performs an atomic $set on the field.             | person.set(name: "Tyler Durden") | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$set" => { name: "Tyler Durden" })
`Model#unset`Performs an atomic $unset on the field.              | person.unset(:name) | collections[:people].find(...).<br/> &nbsp;&nbsp;update("$unset" => { name: 1 })

## Custom
----

There maybe cases where you want to persist documents to different sources from their defaults, or with different options from the default. Mongoid provides run-time support for this as well as support on a per-model basis.
Model Level Persistence Options

On a per-model basis, you can tell it to store in a custom collection name, a different database, or a different session. The following example would store the Band class by default into a collection named "artists" in the database named "music", with the session "secondary".
	Note that the value supplied to the session option must be configured under sessions in your mongoid.yml.

classBand include Mongoid::Document store_in collection: "artists", database: "music", session: "secondary"end

If no `store_in` macro would have been provided, Mongoid would store the model in a collection named "bands" in the default database in the default session.
Runtime Persistence Options

You can change at runtime where to store, query, update, or remove documents by prefixing any operation with #with.

Band.with(database: "music-non-stop").createBand.with(collection: "artists").delete_allband.with(session: :tertiary).save!

Persisting using with is a one time switch in the persistence context - it changes back to its defaults immediately after. Mongoid will not remember anything specific on the document level regarding how it was saved when using this method. A potential gotcha with this is persisting a document via with and then immediately updating it after.

band = Band.new(name: "Scuba")band.with(collection: "artists").save!band.update_attribute(likes: 1000) # This will not save - tries the collection "bands"band.with(collection: "artists").update_attribute(likes: 1000) # This will update the document.

If you want to switch the persistence context for all operations at runtime, but don't want to be using with all over your code, Mongoid provides the ability to do this as the session and database level globally. The methods for this are Mongoid.override_session and Mongoid.override_database. A useful case for this are internationalized applications that store information for different locales in different databases or sessions, but the schema in each remains the same.

classBandsController <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >ApplicationController before_filter :switch_database after_filter :reset_database private defswitch_databaseI18n.locale = params[:locale] || I18n.default_locale Mongoid.override_database("my_db_name_#{I18n.locale}") enddefreset_databaseMongoid.override_database(nil) endend

In the above example, all persistence operations would be stored in the alternative database for all remaining operations on this thread. This is why the after request set the override back to nil - it ensures subsequent requests with no local params use the default option.

#withis also used for changing safe mode options temporarily at runtime.

Band.with(safe: true).createBand.with(safe: { w: 3 }).create!

The values that can be passed with :safe are:

    true: Persist in safe mode, no extra options.
    false: Don't persist in safe mode.
    fsync: true|false: Whether to perform an fsync.
    w: n: The number of nodes to write to before returning.
    wtimeout: n: The timeout value for writing to multiple nodes.

Session and Collection Access

If you want to drop down to the driver level to perform operations, you can grab the Moped session or collection from the model or document instance.

Band.mongo_sessionband.mongo_sessionBand.collectionband.collection

From here you also have the same runtime persistence options using Moped's #with. The Moped documentation will go into more detail about this.

Band.mongo_session.with(safe: false, database: "musik") do |session|  session[:artists].find(...)end

Capped Collections

Mongoid does not provide a mechanism for creating capped collections on the fly - you will need to create these yourself one time up front either with Moped or via the Mongo console.

To create a capped collection with Moped:

session.command(create: "name", capped: true, size: 10000000, max: 1000)

To create a capped collection from the Mongo console:

db.createCollection("name", { capped: true, size: 10000000, max: 1000 });

