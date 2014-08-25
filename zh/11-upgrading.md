---
layout: post
title: 
---


Upgrading

Use this as a reference when upgrading between Mongoid versions. You can always reference the CHANGELOG for bug fixes, new features, and major changes between versions.
	Mongoid follows versioning guidelines as outlined by the Semantic Versioning Specification, so you can expect only backwards incompatible changes in major versions.
Upgrading to 3.0

If you are just wanting to generate a new mongoid.yml from scratch, ensure you delete the old one first so you do not get validation errors when initializing the app at generation time.

The following are backwards incompatible changes in this release.

    bson and bson_ext gems are no longer necessary, and `BSON::ObjectId` objects are no longer compatible with Mongoid documents. Use Moped::BSON::ObjectId instead. It is safe to remove both gems from your Gemfile and any other dependencies. They have been replaced by Moped (see Moped::BSON). Note that if you had not been converting your object ids to strings and were storing them in any way that would cause a Marshal dump, you may need to restore these values as strings. This is because the internals of Moped's BSON and the 10gen BSON gem are not designed in the same manner, even though both have the same public API and adhere to the same spec.
    GridFS support no longer exists. If you are using GridFS, you cannot upgrade.
    Indexing syntax has changed. The first parameter is now a hash of name/direction pairs with an optional second hash parameter for additional options.

    Normal indexing with options, directions are either 1 or -1:

    class Band
      include Mongoid::Document
      field :name, type: String
      index({ name: 1 }, { unique: true, background: true })
    end

    Geospatial indexing needs "2d" as its direction.

    class Venue
      include Mongoid::Document
      field :location, type: Array

      index location: "2d"
    end

    Custom serializable fields have revamped. Your object no longer should include Mongoid::Fields::Serializable - instead it only needs to implement 4 methods: #mongoize, .mongoize, .demongoize and .evolve.

    See Custom Fields Serialization for details on the implementation.
    Document#changes is no longer a hash with indifferent access.
    after_initialize callbacks no longer cascade to children if the option is set.
    #1865 count on the memory and mongo contexts now behave exactly the same as Ruby's count on enumerable, and can take an object or a block. This is optimized on the mongo context not to load everything in memory with the exception of passing a block.

    Band.where(name: "Tool").count
    Band.where(name: "Tool").count(tool) # redundant.
    Band.where(name: "Tool") do |doc|
      doc.likes > 0
    end

    Note that although the signatures are the same for both the memory and mongo contexts, it's recommended you only use the block syntax for the memory context since the embedded documents are already loaded into memory.

    Also note that passing a boolean to take skip and limit into account is no longer supported, as this is not necessarily a useful feature.
    The autocreate_indexes configuration option has been removed.
    Model.defaults no longer exists. You may get all defaults with a combination of Model.pre_processed_defaults and Model.post_processed_defaults.

    Band.pre_processed_defaults
    Band.post_processed_defaults

    Model.identity and Model.key have been removed. For custom ids, users must now override the _id field.

    When the default value is a proc, the default is applied *after* all other attributes are set.

    class Band
      include Mongoid::Document
      field :_id, type: String, default: ->{ name }
    end

    To have the default applied before other attributes, set :pre_processed to true.

    class Band
      include Mongoid::Document
      field :_id,
        type: String,
        pre_processed: true,
        default: ->{ Moped::BSON::ObjectId.new.to_s }
    end

    Custom application exceptions in various languages has been removed, along with the Mongoid.add_language feature.
    Mongoid no longer supports 1.8 syntax. 1.9.x or other vms running in 1.9 mode is now only supported.
    #1734 When searching for documents via Model.find with multiple ids, Mongoid will raise an error if not all ids are found, and tell you what the missing ones were. Previously the error only got raised if nothing was returned.
    #1675 Adding presence validation on a relation now enables autosave. This is to ensure that when a new parent object is saved with a new child and marked is valid, both are persisted to ensure a correct state in the database.
    #1484 Model#has_attribute? now behaves the same as Active Record.
    #1475 Active Support's time zone is now used by default in time serialization if it is defined.
    #1471 Mongoid no longer strips any level of precision off of times.
    #1342 Model.find and model.relation.find now only take a single or multiple ids. Model.first, Model.last also no longer take arguments. For these use Model.find_by instead.
    #1291 The mongoid.yml has been revamped completely, and upgrading existing applications will greet you with some lovely Mongoid specific configuration errors. You can re-generate a new mongoid.yml via the existing rake task, which is commented to an insane degree to help you with all the configuration possibilities.
    #1291 The persist_in_safe_mode configuration option has been removed. You must now tell a database session in the mongoid.yml whether or not it should persist in safe mode by default.

    production:
      sessions:
        default:
          database: my_app_prod
          hosts:
            - db.app.com:27018
            - db.app.com:27019
          options:
            consistency: :eventual
            safe: true

    #1291 safely and unsafely have been removed. Please now use with to provide safe mode options at runtime.

    Band.with(safe: true).create
    band.with(safe: { w: 3 }).save!
    Band.with(safe: false).create!

    #1270 Relation macros have been changed to match their AR counterparts: only :has_one, :has_many, :has_and_belongs_to_many, and :belongs_to exist now.
    #1268 Model#new? has been removed, developers must now always use Model#new_record?.
    #1182 A reload is no longer required to refresh a relation after setting the value of the foreign key field for it. Note this behaves exactly as Active Record.

    If the id is set, but the document for it has not been persisted, accessing the relation will return empty results.

    If the id is set and its document is persisted, accessing the relation will return the document.

    If the id is set, but the base document is not saved afterwards, then reloading will return the document to its original state.
    #1093 Field serialization strategies have changed on Array, Hash, Integer and Boolean to be more consistent and match AR where appropriate.

    Serialization of arrays calls Array.wrap(object)

    Serialization of hashes calls Hash[object] (to_hash on the object)

    Serialization of integers always returns an int via to_i

    Serialization of booleans defaults to false instead of nil.
    #933 :field.size has been renamed to :field.with_size in criteria for $size not to conflict with Symbol's size method.
    #797 Mongoid scoping code has been completely rewritten, and now matches the Active Record API. With this backwards incompatible change, some methods have been removed or renamed.

    Criteria#as_conditions and Criteria#fuse no longer exist.

    Criteria#merge now only accepts another object that responds to to_criteria.

    Criteria#merge! now merges in another object without creating a new criteria object.

    Band.where(name: "Tool").merge!(criteria)

    Named scopes and default scopes no longer take hashes as parameters. From now on only criteria and procs wrapping criteria will be accepted, and an error will be raised if the arguments are incorrect.

    class Band
      include Mongoid::Document

      default_scope ->{ where(active: true) }
      scope :inactive, where(active: false)
      scope :invalid, where: { valid: false } # This will raise an error.
    end

    The named_scope macro has been removed, from now on only use scope.

    Model.unscoped now accepts a block which will not allow default scoping to be applied for any calls inside the block.

    Band.unscoped do
      Band.scoped.where(name: "Ministry")
    end

    Model.scoped now takes options that will be set directly on the criteria options hash.

    Band.scoped(skip: 10, limit: 20)

