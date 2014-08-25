---
layout: post
title: 嵌套属性 
---
Nested Attributes

Nested attributes provide a mechanism for updating documents and their relations in a single operation, by nesting attributes in a single parameters hash. This is extremely useful when wanting to edit multiple documents within a single web form.

    Common Behaviour
    Operations 

Common Behaviour

Nested attributes can be enabled for any relation, embedded or referenced. To enable this for the relation, simply provide the relation name to the accepts_nested_attributes_for macro.

classBand include Mongoid::Document embeds_many :albums belongs_to :producer accepts_nested_attributes_for :albums, :producerend

	

Note that when you add nested attributes functionality to a referenced relation, Mongoid will automatically enable autosave for that relation.

When a relation gains nested attributes behavior, an additional method is added to the base model, which should be used to update the attributes with the new functionality. This method is the relation name plus _attributes=. You can use this method directly, or more commonly the name of the method can be an attribute in the updates for the base class, in which case Mongoid will call the appropriate setter under the covers.

band = Band.firstband.producer_attributes = { name: "Flood" }band.attributes = { producer_attributes: { name: "Flood" }}

Note that this will work with any attribute based setter method in Mongoid. This includes: update_attributes, update_attributes! and attributes=. For full examples of every single scenario you could possibly imagine, see the One Spec to Rule them All.
Operations

The following table shows the means of setting nested attributes for the different types of relations, as well as the required and optional options.
1-1 Operations

1-1 operations include embeds_one, embedded_in, has_one, and belongs_to.
Operation	Syntax	Definition

Set the relation via nested attributes, or replace the existing one.
	

album.producer_attributes = { name: "Flood" }album.attributes =  { producer_attributes: { name: "Flood" }}

	

classAlbum include Mongoid::Document belongs_to :producer accepts_nested_attributes_for :producerend

Update the existing relation - an id or _id must be provided in this case of the existing document.
	

album.producer_attributes =  { id: ..., name: "Flood" }album.attributes = {  producer_attributes: { id: ..., name: "Flood" }}

	

classAlbum include Mongoid::Document belongs_to :producer accepts_nested_attributes_for :producerend

Reject nested attributes if they don't match a certain criteria.
	

album.producer_attributes = { name: "Flood" }album.attributes = {  producer_attributes: { name: "Flood" }}

	

classAlbum include Mongoid::Document belongs_to :producer accepts_nested_attributes_for :producer,    reject_if: -<(attrs){ attrs[:name] == "Flood" }end

Reject nested attributes if all the fields are blank.
	

album.producer_attributes =  { name: "", label: "" }album.attributes = {  producer_attributes = { name: "", label: "" }}

	

classAlbum include Mongoid::Document belongs_to :producer accepts_nested_attributes_for :producer,    reject_if: :all_blankend

Delete an existing 1-1 relation via nested attributes. The attributes must all match, and a _destroy truthy value must be passed.
	

album.producer_attributes =  { name: "Flood", _destroy: "1" }album.attributes = {  producer_attributes:    { name: "Flood", _destroy: "1" }}

	

classAlbum include Mongoid::Document belongs_to :producer accepts_nested_attributes_for :producer,    allow_destroy: trueend

Update an existing relation, never create a new one - even if the id or _id does not match.
	

album.producer_attributes =  { id: ..., name: "Flood" }album.attributes = {  producer_attributes:    { id: ..., name: "Flood" }}

	

classAlbum include Mongoid::Document belongs_to :producer accepts_nested_attributes_for :producer,    update_only: trueend

1-n/n-n Operations

1-n/n-n operations include embeds_many, has_many and has_and_belongs_to_many. These attribute hashes can take either a hash of hashes, with arbitrary keys for each document, or an array with a hash for each document in it.
Operation	Syntax	Definition

Create a new document on the relation
	

band.albums_attributes = { "0" =< { name: "Violator" }}band.albums_attributes = [  { name: "Violator" }]band.attributes = {  albums_attributes: { "0" =< { name: "Violator" }  }}

	

classBand include Mongoid::Document embeds_many :albums accepts_nested_attributes_for :albumsend

Limit the number of new documents that can be created in a single set, raising an error if more are passed in.
	

band.albums_attributes = { "0" =< { name: "Violator" }, "1" =< { name: "101" }, "2" =< { name: "Music for the Masses" }}band.attributes = {  albums_attributes: { "0" =< { name: "Violator" }, "1" =< { name: "101" }, "2" =< { name: "Music for the Masses" }  }}

	

classBand include Mongoid::Document embeds_many :albums accepts_nested_attributes_for :albums, limit: 2end

Update existing documents in the relation - id or _id must be provided in the attributes for each document.
	

band.albums_attributes = { "0" =< { id: ..., name: "Violator (Edit)" }, "1" =< { id: ..., name: "101 (Edit)" }}band.albums_attributes = [  { id: ..., name: "Violator (Edit)" },  { id: ..., name: "101 (Edit)" }]band.attributes = {  albums_attributes: { "0" =< { id: ..., name: "Violator (Edit)" }, "1" =< { id: ..., name: "101 (Edit)" }  }}

	

classBand include Mongoid::Document embeds_many :albums accepts_nested_attributes_for :albumsend

Delete a document in the relation - id or _id must be passed in along with the _destroy truthy value.
	

band.albums_attributes = { "0" =< { id: ..., _destroy: "1" },}band.albums_attributes = [  { id: ..., _destroy: "1" },]band.attributes = {  albums_attributes: { "0" =< { id: ..., _destroy: "1" },  }}

	

classBand include Mongoid::Document embeds_many :albums accepts_nested_attributes_for :albums,    allow_destroy: trueend

Reject nested attributes if they don't match a certain criteria.
	

band.albums_attributes = { "0" =< { name: "Violator" }}band.albums_attributes = [  { name: "Violator" }]band.attributes = {  albums_attributes: { "0" =< { name: "Violator" }  }}

	

classBand include Mongoid::Document embeds_many :albums accepts_nested_attributes_for :albums,    reject_if: -<(attrs){ attrs[:name] == "Violator" }end

Reject nested attributes if all the fields are blank.
	

band.albums_attributes = { "0" =< { name: "" }}band.albums_attributes = [  { name: "" }]band.attributes = {  albums_attributes: { "0" =< { name: "" }  }}

	

classBand include Mongoid::Document embeds_many :albums accepts_nested_attributes_for :albums,    reject_if: :all_blankend


