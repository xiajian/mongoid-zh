---
layout: post
title: 关系 
---

Relations

Relations are associations between one model and another in the domain and in the database. Embedded relations describe documents who are stored inside other documents in the database. Referenced relations describe documents that reference documents in another collection by storing foreign key data (usually an id) about the other document in itself.

All relation objects in Mongoid are proxies to the actual document or documents themselves, which provide extra functionality for accessing, replacing, appending and persisting.

    Common Behaviour
    Metadata
    Embedded 1-1
    Embedded 1-n
    Referenced 1-1
    Referenced 1-n
    Referenced n-n 

Common Behaviour
Attributes

All relations contain a target, which is the proxied document or documents, a base which is the document the relation hangs off, and metadata which provides information about the relation.

classPerson include Mongoid::Document embeds_many :addressesendperson.addresses = [ address ]person.addresses.target # returns [ address ]person.addresses.base # returns personperson.addresses.metadata # returns the metadata

Extensions

All relations can have extensions, which provides a way to add application specific functionality to the relation. They are defined by providing a block to the relation definition.

classPerson include Mongoid::Document embeds_many :addressesdodeffind_by_country(country)      where(country: country).first enddefchinese@target.select { |address| address.country == "China"} endendendperson.addresses.find_by_country("Mongolia") # returns addressperson.addresses.chinese # returns [ address ]

Custom Relation Names

You can name your relations whatever you like, but if the class cannot be inferred by Mongoid from the name, and neither can the opposite side you'll want to provide the macro with some additional options to tell Mongoid how to hook them up.

classLush include Mongoid::Document embeds_one :whiskey, class_name: "Drink", inverse_of: :alcoholicendclassDrink include Mongoid::Document embedded_in :alcoholic, class_name: "Lush", inverse_of: :whiskeyend

Validations

It is important to note that by default, Mongoid will validate the children of any relation that are loaded into memory via a validates_associated. The relations that this applies to are:

    embeds_many
    embeds_one
    has_many
    has_one
    has_and_belongs_to_many 

If you do not want this behavior, you may turn it off when defining the relation.

classPerson include Mongoid::Document embeds_many :addresses, validate: false has_many :posts, validate: falseend

Polymorphism

When a child embedded document can belong to more than one type of parent document, you can tell Mongoid to support this by adding the as option to the definition on the parents, and the polymorphic option on the child. On the child object, an additional field will be stored that indicates the type of the parent.
	

Polymorphic behavior is allowed on all relations with the exception of has_and_belongs_to_many.

classBand include Mongoid::Document embeds_many :photos, as: :photographic has_one :address, as: :addressableendclassPhoto include Mongoid::Document embedded_in :photographic, polymorphic: trueendclassAddress include Mongoid::Document belongs_to :addressable, polymorphic: trueend

Cascading Callbacks

If you want the embedded document callbacks to fire when calling a persistence operation on its parent, you will need to provide the cascade callbacks option to the relation.
	

Cascading callbacks is only available on embeds_one and embeds_many relations.

classBand include Mongoid::Document embeds_many :albums, cascade_callbacks: true embeds_one :label, cascade_callbacks: trueendband.save # Fires all save callbacks on the band, albums, and label.

Dependent Behaviour

You can provided dependent options to referenced associations to instruct Mongoid how to handle situations where one side of the relation is deleted, or is attempted to be deleted. The options are as follows:

    :delete: Delete the child document.
    :destroy: Destroy the child document.
    :nullify: Orphan the child document.
    :restrict: Raise an error if the child is not empty.

	

Dependent options are only available on referenced relations.

The default behavior of each association when no dependent option is provided is to nullify.

classBand include Mongoid::Document has_many :albums, dependent: :delete belongs_to :label, dependent: :nullifyendclassAlbum include Mongoid::Document belongs_to :bandendclassLabel include Mongoid::Document has_many :bands, dependent: :restrictendlabel = Label.firstlabel.bands.push(Band.first)label.delete # Raises an error since bands is not empty.Band.first.delete # Will delete all associated albums.

Autosaving

One core difference between Mongoid and Active Record from a behavior standpoint is that Mongoid does not automatically save child relations for relational associations. This is for performance reasons.

To enable an autosave on a relational association (embedded associations do not need this since they are actually part of the parent in the database) add the autosave option to the relation.
	Note that autosave functionality will automatically be added to a relation when using accepts_nested_attributes_for or validating presence of the relation.

classBand include Mongoid::Document has_many :albums, autosave: trueendband = Band.firstband.albums.build(name: "101")band.save #=< Will save the album as well.

Recursive Embedding

A document can recursively embed itself using recursively_embeds_one or recursively_embeds_many, which provides accessors for the parent and children via parent_ and child_ methods.
	

Recursive options are only available on embedded relations.

classTag include Mongoid::Document recursively_embeds_manyendroot = Tag.new(name: "programming")child_one = root.child_tags.buildchild_two = root.child_tags.buildroot.child_tags # [ child_one, child_two ]child_one.parent_tag # [ root ]child_two.parent_tag # [ root ]classNode include Mongoid::Document recursively_embeds_oneendroot = Node.newchild = Node.newroot.child_node = childroot.child # childchild.parent_node # root

Existence Predicates

All relations have existence predicates on them in the form of name? and has_name? to check if the relation is blank.

classBand include Mongoid::Document embeds_one :label embeds_many :albumsendband.label?band.has_label?band.albums?band.has_albums?

Autobuilding

One to one relations (embeds_one, has_one) have an autobuild option which tells Mongoid to instantiate a new document when the relation is accessed and it is nil
	

Existence predicates will not trigger an autobuild, so they will properly return false if the document is not present.

classBand include Mongoid::Document embeds_one :label, autobuild: true has_one :producer, autobuild: trueendband = Band.newband.label # Returns a new empty label.band.producer # Returns a new empty producer.

Touching

Any belongs_to relation, no matter where it hangs off from, can take an optional :touch option which will call the touch method on it and any parent relations with the option defined when the base document calls #touch.

classBand include Mongoid::Document belongs_to :label, touch: trueendband = Band.firstband.touch #=< Calls touch on the parent label.

Metadata

All relations in Mongoid contain metadata that holds information about the relation in question, and is a valuable tool for third party developers to use to extend Mongoid.

You can access the metadata of the relation in a few different ways.

# Get the metadata for a named relation from the class or document.Model.reflect_on_association(:relation_name)model.reflect_on_association(:relation_name)# Get the metadata that the current object has in its relation.model.metadata# Get the metadata with a specific relation itself on a specific# document.person.addresses.metadata

The Metadata Object

The metadata object itself contains more information than one might know what to do with, and is useful for developers of extensions to Mongoid.
Method	Description
Metadata#as 	Returns the name of the parent to a polymorphic child.
Metadata#as? 	Returns whether or not an as option exists.
Metadata#autobuilding? 	Returns whether or not the relation is autobuilding.
Metadata#autosaving? 	Returns whether or not the relation is autosaving.
Metadata#cascading_callbacks? 	Returns whether the relation has callbacks cascaded down from the parent.
Metadata#class_name 	Returns the class name of the proxied document.
Metadata#cyclic? 	Returns whether the relation is a cyclic relation.
Metadata#dependent 	Returns the relation's dependent option.
Metadata#dependent? 	Returns whether the relation is a dependent relation.
Metadata#destructive? 	Returns true if the relation has a dependent delete or destroy.
Metadata#embedded? 	Returns whether the relation is embedded in another document.
Metadata#forced_nil_inverse? 	Returns whether the relation has a nil inverse defined.
Metadata#foreign_key 	Returns the name of the foreign key field.
Metadata#foreign_key_check 	Returns the name of the foreign key field dirty check method.
Metadata#foreign_key_setter 	Returns the name of the foreign key field setter.
Metadata#indexed? 	Returns whether the foreign key is auto indexed.
Metadata#inverses 	Returns the names of all inverse relation.
Metadata#inverse 	Returns the name of a single inverse relation.
Metadata#inverse_class_name 	Returns the class name of the relation on the inverse side.
Metadata#inverse_foreign_key 	Returns the name of the foreign key field on the inverse side.
Metadata#inverse_klass 	Returns the class of the relation on the inverse side.
Metadata#inverse_metadata 	Returns the metadata of the relation on the inverse side.
Metadata#inverse_of 	Returns the explicitly defined name of the inverse relation.
Metadata#inverse_of? 	Returns whether an inverse_of option is defined.
Metadata#inverse_setter 	Returns the name of the method used to set the inverse.
Metadata#inverse_type 	Returns the name for the polymorphic type field of the inverse.
Metadata#inverse_type_setter 	Returns the name for the polymorphic type field setter of the inverse.
Metadata#key 	Returns the name of the field in the attributes hash to use to get the relation.
Metadata#klass 	Returns the class of the proxied documents in the relation.
Metadata#macro 	Returns the relation's macro.
Metadata#name 	Returns the relation name.
Metadata#options 	Returns self, for API compatibility with Active Record.
Metadata#order 	Returns the custom sorting options on the relation.
Metadata#order? 	Returns whether custom sorting options are set.
Metadata#polymorphic? 	Returns whether the relation is polymorphic.
Metadata#setter 	Returns the name of the field to set the relation.
Metadata#store_as 	Returns the name of the attribute to store an embedded relation in.
Metadata#touchable? 	Returns whether or not the relation has a touch option.
Metadata#type 	Returns the name of the field to get the polymorphic type.
Metadata#type_setter 	Returns the name of the field to set the polymorphic type.
Metadata#validate? 	Returns whether the relation has an associated validation.
Metadata#versioned? 	Returns whether the relation is an embedded version.
Embedded 1-1

One to one relationships where the children are embedded in the parent document are defined using Mongoid's embeds_one and embedded_in macros.
Defining

The parent document of the relation should use the embeds_one macro to indicate is has 1 embedded child, where the document that is embedded uses embedded_in.

classBand include Mongoid::Document embeds_one :labelendclassLabel include Mongoid::Document field :name, type: String embedded_in :bandend

	

Definitions are required on both sides to the relation in order for it to work properly.
Storage

Documents that are embedded using the embeds_one macro are stored as a hash inside the parent in the parent's database collection.

{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9"), "label" : { "_id" : ObjectId("4d3ed089fb60ab534684b7e0"), "name" : "Mute",  }}

You can optionally tell Mongoid to store the embedded document in a different attribute other than the name, by providing a :store_as option.

classBand include Mongoid::Document embeds_one :label, store_as: "lab"end

Operations

Once the relation is defined, the following operations are available, and the following table shows any database operations that are performed if applicable. The previously defined models will be used for example code.
Operation	Mongoid	Moped
Model#{name}

Get the embedded document.
	

band.label

Model#{name}=

Set the embedded document. If the parent document is persisted, then the child will be atomically saved immediately. If setting to nil then the child will be deleted.
	

band.label = Label.new(name: "Mute")band.label = nil

	

collections[:bands].find(...).  update( "$set" =< { label: { name: "Mute" }}  )collections[:bands].find(...).  update("$unset" =< { label: true })

Model#{parent_name}

Get the parent document from the child.
	

label.band

Model#{parent_name}=

Set the parent document from the child.
	

label.band = Band.new

Model#build_{name}

Build a new document on the relation.
	

band.build_label(name: "Mute")label.build_band(name: "Depeche Mode")

Model#create_{name}

Create a new document from either side of the relation. This persists the child immediately if executing from the parent, and persists the entire tree if executed from the child.
	

band.create_label(name: "Mute")label.create_band({  name: "Depeche Mode"})

	

collections[:bands].find(...).  update( "$set" =< { label: { name: "Mute" }}  )collections[:bands].  insert({    name: "Depeche Mode",    label: { name: "Mute" }  })

Embedded 1-n

One to many relationships where the children are embedded in the parent document are defined using Mongoid's embeds_many and embedded_in macros.
Defining

The parent document of the relation should use the embeds_many macro to indicate it has n number of embedded children, where the document that is embedded uses embedded_in.

classBand include Mongoid::Document embeds_many :albumsendclassAlbum include Mongoid::Document field :name, type: String embedded_in :bandend

	

Definitions are required on both sides to the relation in order for it to work properly.
Storage

Documents that are embedded using the embeds_many macro are stored as an array of hashes inside the parent in the parent's database collection.

{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9"), "albums" : [    { "_id" : ObjectId("4d3ed089fb60ab534684b7e0"), "name" : "Violator",    }  ]}

You can optionally tell Mongoid to store the embedded document in a different attribute other than the name, by providing a :store_as option.

classBand include Mongoid::Document embeds_many :albums, store_as: "albs"end

Operations

Once the relation is defined, the following operations are available, and the following table shows any database operations that are performed if applicable. The previously defined models will be used for example code.
Operation	Mongoid	Moped
Model#{name}

Get the embedded documents.
	

band.albums

Model#{name}=

Set the embedded documents. If the parent document is persisted, then the child will be atomically saved immediately. If setting to nil or [] then the children will be deleted.
	

band.albums = [ Album.new(name: "Violator") ]band.albums = nilband.albums = []

	

collections[:bands].find(...).  update( "$set" =< { albums: [{ name: "Violator" }]}  )collections[:bands].find(...).  update("$unset" =< { albums: true })

Model#{parent_name}

Get the parent document from any child.
	

album.band

Model#{parent_name}=

Set the parent document from a child.
	

album.band = Band.new

Model#{name}.< code>
Model#{name}.push

Push a new document onto the relation. If the parent is persisted, then the child documents will be automatically saved.
	

band.albums <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Album.new(name: "Violator")band.albums.push(Album.new(name: "Violator"))

	

collections[:bands].find(...).  update( "$push" =< { albums: { name: "Violator" }}  )

Model#{name}.concat

Push multiple documents onto the relation. If the parent is persisted, then the child documents will be automatically saved. Note that while batch operations limit the number of database calls to a single one for the new documents, it is in fact at least 2x slower from the Ruby side. This is due to the fact that 2 iterations over all documents must occur to ensure that all the before callbacks run before the db hit, and that all after callbacks have to wait until after.
	

band.albums.concat( Album.new(name: "Violator"), Album.new(name: "101"))

	

collections[:bands].find(...).  update( "$pushAll" =< {      albums: [{ name: "Violator" }, { name: "101" }]    }  )

Model#{name}.build
Model#{name}.new

Build a new document in the relation with the provided attributes. Does not save the new document.
	

band.albums.build(name: "Violator")band.albums.new(name: "Violator")

Model#{name}.create
Model#{name}.create!

Create a new document in the relation with the provided attributes and saves. With the bang version an error will be raised if validation fails.
	

band.albums.create(name: "Violator")band.albums.create!(name: "Violator")

	

collections[:bands].find(...).  update( "$push" =< { albums: { name: "Violator" }}  )

Model#{name}.clear
Model#{name}.delete_all

Deletes all documents from the relation, without running any callbacks.
	

band.albums.clearband.albums.delete_allband.albums.delete_all(name: "Violator")band.albums.  where(name: "Violator").delete_all

	

collections[:bands].find(...).  update( "$pullAll" =< { albums: [{ name: "Violator" }]}  )

Model#{name}.destroy_all

Deletes all documents from the relation, while running the destroy callbacks.
	

band.albums.destroy_allband.albums.destroy_all(name: "Violator")band.albums.  where(name: "Violator").destroy_all

	

collections[:bands].find(...).  update( "$pullAll" =< { albums: [{ name: "Violator" }]}  )

Model#{name}.delete

Deletes the matching document from the relation.
	

band.albums.delete(album)

	

collections[:bands].find(...).  update( "$pull" =< { albums: { name: "Violator" }}  )

Model#{name}.pop

Deletes the provided number of documents, defaulting to 1.
	

band.albums.popband.albums.pop(1)

	

collections[:bands].find(...).  update( "$pullAll" =< { albums: [{ name: "Violator" }]}  )

Model#{name}.find

Return documents in the relation with matching ids. Will raise an error if all the ids are not found by default.
	

band.albums.find(id)band.albums.find(id_one, id_two)

Model#{name}.find_or_create_by

Search for the document in the relation, and if not found create a newly persisted one.
	

band.albums.  find_or_create_by(name: "Violator")

	

collections[:bands].find(...).  update( "$push" =< { albums: { name: "Violator" }}  )

Model#{name}.find_or_initialize_by

Search for the document in the relation, and if not found add a new one.
	

band.albums.  find_or_initialize_by(name: "Violator")

Model#{name}.where

Find matching documents in the relation. This can be any criteria method, not just where.
	

band.albums.where(name: "Violator")

Model#{name}.exists?

Returns whether or not the relation has any documents.
	

band.albums.exists?

Referenced 1-1

One to one relationships where the children are referenced in the parent document are defined using Mongoid's has_one and belongs_to macros.
Defining

The parent document of the relation should use the has_one macro to indicate is has 1 referenced child, where the document that is referenced in it uses belongs_to.

classBand include Mongoid::Document has_one :studioendclassStudio include Mongoid::Document field :name, type: String belongs_to :bandend

Definitions are required on both sides to the relation in order for it to work properly, unless one of the models is embedded.
Storage

When defining a relation of this nature, each document is stored in its respective collection, but the child document contains a "foreign key" reference to the parent.

# The parent band document.{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9") }# The child studio document.{ "_id" : ObjectId("4d3ed089fb60ab534684b7f1"), "band_id" : ObjectId("4d3ed089fb60ab534684b7e9")}

Operations

Once the relation is defined, the following operations are available, and the following table shows any database operations that are performed if applicable. The previously defined models will be used for example code.
Operation	Mongoid	Moped
Model#{name}

Get the child document.
	

band.studio

Model#{name}=

Set the child document. If the parent document is persisted, then the child will be saved immediately. If setting to nil then the child will be deleted.
	

band.studio = Studio.new(name: "Abbey Road")band.studio = nil

	

collections[:studios].insert(  { name: "Abbey Road", band_id: ... })collections[:studios].find(...).remove

Model#{parent_name}

Get the parent document from the child.
	

studio.band

Model#{parent_name}=

Set the parent document from the child.
	

studio.band = Band.new

Model#build_{name}

Build a new document on the relation. This does not save the new document.
	

band.build_studio(name: "Abbey Road")studio.build_band(name: "Depeche Mode")

Model#create_{name}

Create a new document from either side of the relation. This persists the child immediately if executing from the parent, and persists the parent if executed from the child.
	

band.create_studio(name: "Abbey Road")studio.create_band(name: "Depeche Mode")

	

collections[:studios].insert(  { name: "Abbey Road", band_id: ... })collections[:bands].insert(  { name: "Depeche Mode" })

Referenced 1-n

One to many relationships where the children are stored in a separate collection from the parent document are defined using Mongoid's has_many and belongs_to macros. This exhibits similar behavior to Active Record.
Defining

The parent document of the relation should use the has_many macro to indicate is has n number of referenced children, where the document that is referenced uses belongs_to.

classBand include Mongoid::Document has_many :membersendclassMember include Mongoid::Document field :name, type: String belongs_to :bandend

Definitions are required on both sides to the relation in order for it to work properly, unless one of the models is embedded.
Storage

When defining a relation of this nature, each document is stored in its respective collection, but the child document contains a "foreign key" reference to the parent.

# The parent band document.{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9") }# The child member document.{ "_id" : ObjectId("4d3ed089fb60ab534684b7f1"), "band_id" : ObjectId("4d3ed089fb60ab534684b7e9")}

Operations

Once the relation is defined, the following operations are available, and the following table shows any database operations that are performed if applicable. The previously defined models will be used for example code.
Operation	Mongoid	Moped
Model#{name}

Get the related documents.
	

band.members

Model#{name}=

Set the related documents. If the parent document is persisted, then the child will be saved immediately. If setting to nil or [] then the children will be deleted.
	

band.members = [ Member.new(name: "Fletch") ]band.members = nilband.members = []

	

collections[:members].insert(  { name: "Fletch", band_id: ... })collections[:members].find(...).remove

Model#{name}_ids

Get the related document ids.
	

band.member_ids

Model#{name}_ids=

Set the related document ids.
	

band.member_ids = [ id ]

Model#{parent_name}

Get the parent document from any child.
	

member.band

Model#{parent_name}=

Set the parent document from a child.
	

member.band = Band.new

Model#{name}.< code>
Model#{name}.push

Push a new document onto the relation. If the parent is persisted, then the child documents will be automatically saved.
	

band.members <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Member.new(name: "Fletch")band.members.push(Member.new(name: "Fletch"))

	

collections[:members].insert(  { name: "Fletch", band_id: ... })

Model#{name}.concat

Push multiple documents onto the relation. If the parent is persisted, then the child documents will be automatically saved in a single batch. Note that while batch operations limit the number of database calls to a single one for the new documents, it is in fact at least 2x slower from the Ruby side. This is due to the fact that 2 iterations over all documents must occur to ensure that all the before callbacks run before the db hit, and that all after callbacks have to wait until after.
	

band.members.concat( Member.new(name: "Fletch"), Member.new(name: "Martin"))

	

collections[:members].insert([  { name: "Fletch", band_id: ... },  { name: "Martin", band_id: ... }])

Model#{name}.build
Model#{name}.new

Build a new document in the relation with the provided attributes. Does not save the new document.
	

band.members.build(name: "Fletch")band.members.new(name: "Fletch")

Model#{name}.create
Model#{name}.create!

Create a new document in the relation with the provided attributes and saves. With the bang version an error will be raised if validation fails.
	

band.members.create(name: "Fletch")band.members.create!(name: "Fletch")

	

collections[:members].insert(  { name: "Fletch", band_id: ... })

Model#{name}.clear
Model#{name}.delete_all

Deletes all documents from the relation, without running any callbacks.
	

band.members.clearband.members.delete_allband.members.delete_all(name: "Fletch")band.members.  where(name: "Fletch").delete_all

	

collections[:members].find(band_id: ...).remove_allcollections[:members].  find(band_id: ..., name: "Fletch").remove_all

Model#{name}.destroy_all

Deletes all documents from the relation, while running the destroy callbacks.
	

band.members.destroy_allband.members.destroy_all(name: "Fletch")band.members.  where(name: "Fletch").destroy_all

	

collections[:members].find(band_id: ...).remove_allcollections[:members].  find(band_id: ..., name: "Fletch").remove_all

Model#{name}.delete

Deletes the matching document from the relation.
	

band.members.delete(member)

	

collections[:members].  find(band_id: ..., name: "Fletch").remove

Model#{name}.find

Return documents in the relation with matching ids. Will raise an error if all the ids are not found by default.
	

band.members.find(id)band.members.find(id_one, id_two)

	

collections[:members].find({  band_id: ..., _id: id})collections[:members].find({  band_id: ..., _id: { "$in" =< [ id_one, id_two ] }})

Model#{name}.find_or_create_by

Search for the document in the relation, and if not found create a newly persisted one.
	

band.members.  find_or_create_by(name: "Fletch")

	

collections[:members].find({  band_id: ..., name: "Fletch"})collections[:members].insert(  { name: "Fletch", band_id: ... })

Model#{name}.find_or_initialize_by

Search for the document in the relation, and if not found add a new one.
	

band.members.  find_or_initialize_by(name: "Fletch")

	

collections[:members].find({  band_id: ..., name: "Fletch"})

Model#{name}.where

Find matching documents in the relation. This can be any criteria method, not just where.
	

band.members.where(name: "Fletch")

	

collections[:members].find({  band_id: ..., name: "Fletch"})

Model#{name}.exists?

Returns whether or not the relation has any documents.
	

band.members.exists?

	

collections[:members].find(band_id: ...).count

Referenced n-n

Many to many relationships where the inverse documents are stored in a separate collection from the base document are defined using Mongoid's has_and_belongs_to_many macro. This exhibits similar behavior to Active Record with the exception that no join collection is needed, the foreign key ids are stored as arrays on either side of the relation.
Defining

Both sides of the relation use the same macro.

classBand include Mongoid::Document has_and_belongs_to_many :tagsendclassTag include Mongoid::Document field :name, type: String has_and_belongs_to_many :bandsend

You can create a one sided many to many if you want to mimic a has_many that stores the keys as an array on the parent.

classBand include Mongoid::Document has_and_belongs_to_many :tags, inverse_of: nilendclassTag include Mongoid::Document field :name, type: Stringend

Storage

When defining a relation of this nature, each document is stored in its respective collection, and each document contains a "foreign key" reference to the other in the form of an array.

# The band document.{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9"), "tag_ids" : [ ObjectId("4d3ed089fb60ab534684b7f2") ]}# The tag document.{ "_id" : ObjectId("4d3ed089fb60ab534684b7f2"), "band_ids" : [ ObjectId("4d3ed089fb60ab534684b7e9") ]}

Operations

Once the relation is defined, the following operations are available, and the following table shows any database operations that are performed if applicable. The previously defined models will be used for example code.
	

Many to many relations require usually double the amount of hits to the database to keep both sides of the relation in sync, since keys are stored on both sides. Due to this they are slower and should be used with caution.
Operation	Mongoid	Moped
Model#{name}

Get the related documents.
	

band.tags

Model#{name}=

Set the related documents. If the parent document is persisted, then the child will be saved immediately along with the parent to keep the keys consistent. If setting to nil or [] then the children will be deleted.
	

band.tags = [ Tag.new(name: "electro") ]band.tags = nilband.tags = []

	

collections[:tags].insert(  { name: "electro", band_ids: ... })collections[:bands].find(...).  update("$push" =< { tag_ids: ... })collections[:tags].find(...).removecollections[:bands].find(...).  update("$pull" =< { tag_ids: ... })

Model#{name}.< code>
Model#{name}.push

Push a new document onto the relation. If the parent is persisted, then the child documents will be automatically saved.
	

band.tags <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Tag.new(name: "electro")band.tags.push(Tag.new(name: "electro"))

	

collections[:tags].insert(  { name: "electro", band_ids: ... })collections[:bands].find(...).  update("$push" =< { tag_ids: ... })

Model#{name}.concat

Push multiple documents onto the relation. If the parent is persisted, then the child documents will be automatically saved in a single batch. Note that while batch operations limit the number of database calls to a single one for the new documents, it is in fact at least 2x slower from the Ruby side. This is due to the fact that 2 iterations over all documents must occur to ensure that all the before callbacks run before the db hit, and that all after callbacks have to wait until after.
	

band.tags.concat( Tag.new(name: "electro"), Tag.new(name: "new wave"))

	

collections[:tags].insert([  { name: "electro", band_ids: ... },  { name: "new wave", band_ids: ... }])collections[:bands].find(...).  update("$pushAll" =< { tag_ids: ... })

Model#{name}.build
Model#{name}.new

Build a new document in the relation with the provided attributes. Does not save the new document.
	

band.tags.build(name: "electro")band.tags.new(name: "electro")

Model#{name}.create
Model#{name}.create!

Create a new document in the relation with the provided attributes and saves. With the bang version an error will be raised if validation fails.
	

band.tags.create(name: "electro")band.tags.create!(name: "electro")

	

collections[:tags].insert(  { name: "electro", band_ids: ... })collections[:bands].find(...).  update("$push" =< { tag_ids: ... })

Model#{name}.clear
Model#{name}.delete_all

Deletes all documents from the relation, without running any callbacks.
	

band.tags.clearband.tags.delete_allband.tags.delete_all(name: "electro")band.tags.  where(name: "electro").delete_all

	

collections[:tags].find(_id: { "$in" =< ... }).remove_allcollections[:bands].find(...).  update("$pullAll" =< { tag_ids: ... })collections[:tags].  find(_id: { "$in" =< ... }, name: "electro").remove_allcollections[:bands].find(...).  update("$pullAll" =< { tag_ids: ... })

Model#{name}.destroy_all

Deletes all documents from the relation, while running the destroy callbacks.
	

band.tags.destroy_allband.tags.destroy_all(name: "electro")band.tags.  where(name: "electro").destroy_all

	

collections[:tags].find(_id: { "$in" =< ... }).remove_allcollections[:bands].find(...).  update("$pullAll" =< { tag_ids: ... })collections[:tags].  find(_id: { "$in" =< ... }, name: "electro").remove_allcollections[:bands].find(...).  update("$pullAll" =< { tag_ids: ... })

Model#{name}.delete

Deletes the matching document from the relation.
	

band.tags.delete(tag)

	

collections[:tags].  find(_id: { "$in" =< ... }, name: "electro").removecollections[:bands].find(...).  update("$pull" =< { tag_ids: ... })

Model#{name}.find

Return documents in the relation with matching ids. Will raise an error if all the ids are not found by default.
	

band.tags.find(id)band.tags.find(id_one, id_two)

	

collections[:tags].find({  _id: { "$in" =< ... }, { "$and" =< [{ _id: id }] }})collections[:tags].find({  _id: { "$in" =< ... },  { "$and" =< [{ _id: { "$in" =< [ id_one, id_two ]}}] }})

Model#{name}.find_or_create_by

Search for the document in the relation, and if not found create a newly persisted one.
	

band.tags.  find_or_create_by(name: "electro")

	

collections[:tags].find({  _id: { "$in" =< ... }, name: "electro"})collections[:tags].insert(  { name: "electro", band_ids: ... })

Model#{name}.find_or_initialize_by

Search for the document in the relation, and if not found add a new one.
	

band.tags.  find_or_initialize_by(name: "electro")

	

collections[:tags].find({  _id: { "$in" =< ... }, name: "electro"})

Model#{name}.where

Find matching documents in the relation. This can be any criteria method, not just where.
	

band.tags.where(name: "electro")

	

collections[:tags].find({  _id: { "$in" =< ... }, name: "electro"})

Model#{name}.exists?

Returns whether or not the relation has any documents.
	

band.tags.exists?

	

collections[:tags].  find(_id: { "$in" =< ... }).count

