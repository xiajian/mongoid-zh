---
layout: post
title: 文档类型 
---

Documents

Documents are the core objects in Mongoid and any object that is to be persisted to the database must include Mongoid::Document. The representation of a Document in MongoDB is a BSON object that is very similar to a Ruby hash or JSON object. Documents can be stored in their own collections in the database, or can be embedded in other Documents n levels deep.

文档是Mongoid中的核心对象，并且任何需要持久化到数据库中的对象都必须包含Mongoid::Document。文档在MongoDB中的表示为BSON，这类似Ruby的hash表和JSON对象。文档可以保存到数据库的集合中，嵌套到其他的文档中。

    Storage
    Fields
    Dynamic Fields
    Localized Fields
    Dirty Tracking
    Security
    Readonly Attributes
    Inheritance 

Storage

Mongoid by default stores documents in a collection that is the pluralized form of the class name. For the following Person class, the collection the document would get stored in would be named people.

Mongoid默认以类名的复数形式将文档存储在集合中。比如，Person类对应集合中的people中。

class Person  include Mongoid::Document end

Model class name cannot end with "s", because it will be considered as the pluralized form of the word. For example Status would be considered as the plural form of Statu, which will cause a few known problems.

模型的名字不能以s结尾，因为以复数结尾

This is a limitation of the ActiveSupport::Inflector#classify which Mongoid uses to convert from filenames and collection names to class names. You can overcome this by specifying a custom inflection rule for your model class. For example, the following code will take care of the model named Status

ActiveSupport::Inflector.inflections do |inflect|  inflect.singular("status", "status")end

The collection for the model's documents can be changed at the class level if you would like them persisted elsewhere. You can also change the database and session the model gets persisted in from the defaults.

classPerson include Mongoid::Document store_in collection: "citizens", database: "other", session: "secondary"end

The store_in macro can also take lambdas - a common case for this is multi-tenant applications.

classBand include Mongoid::Document store_in database: -<{ Thread.current[:database] }end

When a document is stored in the database the ruby object will get serialized into BSON and have a structure like so:

{ "_id" : ObjectId("4d3ed089fb60ab534684b7e9"), "title" : "Sir", "name" : { "_id" : ObjectId("4d3ed089fb60ab534684b7ff"), "first_name" : "Durran" }, "addresses" : [    { "_id" : ObjectId("4d3ed089fb60ab534684b7e0"), "city" : "Berlin", "country" : "Deutschland" }  ]}

Fields

Even though MongoDB is a schemaless database, most uses will be with web applications where form parameters always come to the server as strings. Mongoid provides an easy mechanism for transforming these strings into their appropriate types through the definition of fields in a Mongoid::Document.

Consider a simple class for modeling a person in an application. A person may have a first name, last name, and middle name. We can define these attributes on a person by using the fields macro.

classPerson include Mongoid::Document field :first_name, type: String field :middle_name, type: String field :last_name, type: Stringend

Below is a list of valid types for fields.

    Array
    BigDecimal
    Boolean
    Date
    DateTime
    Float
    Hash
    Integer
    Moped::BSON::ObjectId
    Moped::BSON::Binary
    Range
    Regexp
    String
    Symbol
    Time
    TimeWithZone 

If you decide not to specify the type of field with the definition, Mongoid will treat it as an object and not try to typecast it when sending the values to the database. This can be advantageous in 2 places, since the lack of attempted conversion will yield a slight performance gain. However some fields are not supported if not defined as fields. A note of thumb for what fields you can use are:

    You're not using a web front end and values are already properly cast.
    All of your fields are strings. 

class Person  include Mongoid::Document  field :first_name  field :middle_name  field :last_name end

Types that are not supported as dynamic attributes since they cannot be cast are:

    BigDecimal
    Date
    DateTime
    Range 

Getting and setting field values

When a field is defined, Mongoid provides several different ways of accessing the field.

# Get the value of the first name field.person.first_nameperson[:first_name]person.read_attribute(:first_name)# Set the value for the first name field.person.first_name = "Jean"person[:first_name] = "Jean"person.write_attribute(:first_name, "Jean")

In cases where you want to set multiple field values at once, there are a few different ways of handling this as well.

# Get the field values as a hash.person.attributes# Set the field values in the document.Person.new(first_name: "Jean-Baptiste", middle_name: "Emmanuel")person.attributes = { first_name: "Jean-Baptiste", middle_name: "Emmanuel" }person.write_attributes(  first_name: "Jean-Baptiste",  middle_name: "Emmanuel")

Hash Field Keys

When using a field of type Hash, be wary of adhering to the legal key names for mongoDB , or else the values will not store properly.

class Person include Mongoid::Document field :first_name field :url, type: Hash# will update the fields properly and save the valuesdefset_valsself.first_name = 'Daniel'self.url = {'home_page' =< 'http://www.homepage.com'}    save end# all data will fail to save due to the illegal hashkeydefset_vals_failself.first_name = 'Daniel'self.url = {'home.page' =< 'http://www.homepage.com'}    save endend

Defaults

You can tell a field in Mongoid to always have a default value if nothing has been provided. Defaults are either static values or lambdas.

class Person include Mongoid::Document field :blood_alcohol_level, type: Float, default: 0.40 field :last_drink, type: Time, default: -<{ 10.minutes.ago } end

Be wary that default values that are not defined as lambdas or procs are evaluated at class load time, so the following 2 definitions are not equivalent. (You probably would prefer the second, which is at document creation time.)

field :dob, type: Time, default: Time.nowfield :dob, type: Time, default: -<{ Time.now }

If you want to set a default with a dependency on the document's state, self inside a lambda or proc evaluates to the document instance.

field :wasted_at, type: Time, default: -<{ new_record? ? 2.hours.ago : Time.now }

	When defining a default value as a proc, Mongoid will apply the default after all other attributes are set. If you want this to happen before the other attributes, set pre_processed: true.
Field Aliasing

One of the drawbacks of having a schemaless database is that MongoDB must store all field information along with every document, meaning that it takes up a lot of storage space in RAM and on disk. A common pattern to limit this is to alias fields to a small number of characters, while keeping the domain in the application expressive. Mongoid allows you to do this and reference the fields in the domain via their long names in getters, setters, and criteria while performing the conversion for you.

class Band include Mongoid::Document field :n, as: :name, type: String end band = Band.new(name: "Placebo")band.attributes #=< { "n" =< "Placebo" }criteria = Band.where(name: "Placebo")criteria.selector #=< { "n" =< "Placebo" }

Custom field serialization

You can define custom types in Mongoid and determine how they are serialized and deserialized. You simply need to provide 3 methods on it for Mongoid to call to convert your object to and from MongoDB friendly values.

class Profile

  include Mongoid::Document

  field :location, type: Point

end

 

class Point

 

  attr_reader :x, :y

 

  def initialize(x, y)

    @x, @y = x, y

  end

 

  # Converts an object of this instance into a database friendly value.

  def mongoize

    [ x, y ]

  end

 

  class << self

 

    # Get the object as it was stored in the database, and instantiate

    # this custom class from it.

    def demongoize(object)

      Point.new(object[0], object[1])

    end

 

    # Takes any possible object and converts it to how it would be

    # stored in the database.

    def mongoize(object)

      case object

      when Point then object.mongoize

      when Hash then Point.new(object[:x], object[:y]).mongoize

      else object

      end

    end

 

    # Converts the object that was supplied to a criteria and converts it

    # into a database friendly form.

    def evolve(object)

      case object

      when Point then object.mongoize

      else object

      end

    end

  end

end

The instance method mongoize take an instance of your object, and converts it into how it will be stored in the database. In our example above, we want to store our point object as an array in the form [ x, y ].

The class method demongoize takes an object as how it was stored in the database, and is responsible for instantiating an object of your custom type. In this case, we take an array and instantiate a Point from it.

The class method mongoize takes an object that you would use to set on your model from your application code, and create the object as it would be stored in the database. This is for cases where you are not passing your model instances of your custom type in the setter:

point = Point.new(12, 24)venue = Venue.new(location: point) #=< This uses the mongoize instance method.venue = Venue.new(location: [ 12, 24 ]) #=< This uses the mongoize class method.

The class method evolve takes an object, and determines how it is to be transformed for use in criteria. For example we may want to write a query like so:

point = Point.new(12, 24)Venue.where(location: point)

	When accessing custom fields from the document, you will get a new instance of that object with each call to the getter. This is because Mongoid is generating a new object from the object from the raw attributes on each access.

We need the point object in the criteria to be transformed to a Mongo friendly value when it is not as well, and evolve is the method that takes care of this. We check if the passed in object is a Point first, in case we also want to be able to pass in ordinary arrays instead.
Reserved names

If you define a field on your document that conflicts with a reserved method name in Mongoid, the configuration will raise an error. For a list of these you may look at Mongoid.destructive_fields.
Creating Custom Ids

For cases when you do not want to have Moped::BSON::ObjectId ids, you can override Mongoid's _id field and set them to whatever you like.

classBand include Mongoid::Document field :name, type: String field :_id, type: String, default: -<{ name }end

Dynamic fields

By default Mongoid supports dynamic fields - that is it will allow attributes to get set and persisted on the document even if a field was not defined for them. There is a slight 'gotcha' however when dealing with dynamic attributes in that Mongoid is not completely lenient about the use of method_missing and breaking the public interface of the Document class.

When dealing with dynamic attributes the following rules apply:

If the attribute exists in the document, Mongoid will provide you with your standard getter and setter methods. For example, consider a person who has an attribute of "gender" set on the document:

# Set the person's gender to male.person[:gender] = "Male"person.gender = "Male"# Get the person's gender.person.gender

If the attribute does not already exist on the document, Mongoid will not provide you with the getters and setters and will enforce normal method_missing behavior. In this case you must use the other provided accessor methods: ([] and []=) or (read_attribute and write_attribute).

# Raise a NoMethodError if value isn't set.person.genderperson.gender = "Male"# Retrieve a dynamic field safely.person[:gender]person.read_attribute(:gender)# Write a dynamic field safely.person[:gender] = "Male"person.write_attribute(:gender, "Male")

Dynamic attributes can be completely turned off by setting the Mongoid configuration option allow_dynamic_fields to false.
Localized fields

From 2.4.0 Mongoid now supports localized fields without the need of any external gem.

classProduct include Mongoid::Document field :description, localize: trueend

By telling the field to localize, Mongoid will under the covers store the field as a hash of locale/value pairs, but normal access to it will behave like a string.

I18n.default_locale = :enproduct = Product.newproduct.description = "Marvelous!"I18n.locale = :deproduct.description = "Fantastisch!"product.attributes#=< { "description" =< { "en" =< "Marvelous!", "de" =< "Fantastisch!" }

You can get and set all the translations at once by using the corresponding _translations method.

product.description_translations#=< { "en" =< "Marvelous!", "de" =< "Fantastisch!" }product.description_translations =  { "en" =< "Marvelous!", "de" =< "Wunderbar!" }

Fallbacks

When using fallbacks, Mongoid will automatically use them when a translation is not available.

For Rails applications, set the fallbacks configuration setting to true in your environment.

config.i18n.fallbacks = true

For non-rails applications, you must include the fallbacks module straight to the I18n gem.

require "i18n/backend/fallbacks"I18n::Backend::Simple.send(:include, I18n::Backend::Fallbacks)

Then when the fallbacks are defined, if a translation is not present Mongoid will fallback in order of the defined locales.

I18n.default_locale = :enI18n.fallbacks = true::I18n.fallbacks[:de] = [ :de, :en, :es ]product = Product.newproduct.description = "Marvelous!"I18n.locale = :deproduct.description #=< "Marvelous!"

Querying

When querying for localized fields using Mongoid's criteria API, Mongoid will automatically alter the criteria to match the current locale.
mongoid

# Match all prodcucts with Marvelous as the description. Locale is en.Product.where(description: "Marvelous!")

mongodb query selector

{ "description.en" : "Marvelous!" }

Indexing

If you plan to be querying extensively on localized fields, you should index each of the locales that you plan on searching on.

class Product include Mongoid::Document field :description, localize: true index "description.de" =< 1 index "description.en" =< 1 end

Validation

Mongoid's presence validator will make sure that translations are present for all locales that are in the underlying hash.
Dirty Tracking

Mongoid supports tracking of changed or "dirty" fields with an API that mirrors that of Active Model. If a defined field has been modified in a model the model will be marked as dirty and some additional behavior comes into play.
Viewing changes

There are various ways to view what has been altered on a model. Changes are recorded from the time a document is instantiated, either as a new document or via loading from the database up to the time it is saved. Any persistence operation clears the changes.

class Person include Mongoid::Document field :name, type: String end person = Person.firstperson.name = "Alan Garner"# Check to see if the document has changed.person.changed? #=< true# Get an array of the names of the changed fields.person.changed #=< [ :name ]# Get a hash of the old and changed values for each field.person.changes #=< { "name" =< [ "Alan Parsons", "Alan Garner" ] }# Check if a specific field has changed.person.name_changed? #=< true# Get the changes for a specific field.person.name_change #=< [ "Alan Parsons", "Alan Garner" ]# Get the previous value for a field.person.name_was #=< "Alan Parsons"

Resetting changes

You can reset changes of a field to its previous value by calling the reset method.

person = Person.firstperson.name = "Alan Garner"# Reset the changed name back to the originalperson.reset_name!person.name #=< "Alan Parsons"

Notes on persistence

Mongoid uses dirty tracking as the core of its persistence operations. It looks at the changes on a document and atomically updates only what has changed unlike other frameworks that write the entire document on each save. If no changes have been made, Mongoid will not hit the database on a call to Model#save.
Viewing previous changes

After a document has been persisted, you can see what the changes were previously by calling Model#previous_changes

person = Person.firstperson.name = "Alan Garner"person.save #=< Clears out current changes.# View the previous changes.person.previous_changes #=< { "name" =< [ "Alan Parsons", "Alan Garner" ] }

Security

There are cases where you don't want Mongoid to allow attributes to be set through mass assignment, like passwords. This is a common event when submitting forms, and fields can be protected by using Mongoid's attr_protected or attr_accessible thanks to the wonders of Active Model.
	Mongoid auto-protects the _id and _type attributes by default.
Protected

When defining a list of fields as protected, all other fields in the document will NOT be able to be set through mass assignment.

classUser include Mongoid::Document field :first_name, type: String field :password, type: String attr_protected :passwordend# Set attributes on a person properly.Person.new(first_name: "Corbin")person.attributes = { first_name: "Corbin" }person.write_attributes(first_name: "Corbin")# Attempt to set attributes a person, logging an error.Person.new(first_name: "Corbin", password: "password")person.attributes = { first_name: "Corbin", password: "password" }person.write_attributes(first_name: "Corbin", password: "password")

Accessible

Providing a list of fields as accessible is simply the inverse of protecting them. Anything not defined as accessible will cause the error.

classUser include Mongoid::Document field :first_name, type: String field :password, type: String attr_accessible :first_nameend# Set attributes on a user properly.User.new(first_name: "Corbin")user.attributes = { first_name: "Corbin" }user.write_attributes(first_name: "Corbin")# Attempt to set attributes on a user, will silently ignore protected ones.User.new(first_name: "Corbin", password: "password")user.attributes = { first_name: "Corbin", password: "password" }user.write_attributes(first_name: "Corbin", password: "password")

You can scope the mass assignment by role by providing the role as an option to the constructor or create methods.

classUser include Mongoid::Document field :first_name, type: String field :password, type: String attr_accessible :first_name, as: [ :default, :admin ]end# Set attributes on a person for the admin rolePerson.new({ first_name: "Corbin" }, as: :admin)Person.create({ first_name: "Corbin" }, as: :default)Person.create!({ first_name: "Corbin" }, as: :admin)

Overriding

In the case you want to override the security in a single call, you can pass a block to the document constructor to set fields manually.

Person.new(first_name: "Corbin") do |person|  person.password = "password"end

Readonly Attributes

You can tell Mongoid that certain attributes are readonly. This will allow documents to be created with theses attributes, but changes to them will be filtered out.

classBand include Mongoid::Document field :name, type: String field :origin, type: String attr_readonly :name, :originendband = Band.create(name: "Placebo")band.update_attributes(name: "Tool") # Filters out the name change.

If you explicitly try to update or remove the attribute by itself, then a ReadonlyAttribute error will be raised.

band.update_attribute(:name, "Tool") # Raises the error.band.remove_attribute(:name) # Raises the error.

Inheritance

Mongoid supports inheritance in both root and embedded documents. In scenarios where documents are inherited from their fields, relations, validations and scopes get copied down into their child documents, but not vise-versa.

classCanvas include Mongoid::Document field :name, type: String embeds_many :shapesendclassBrowser <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Canvas field :version, type: Integer scope :recent, where(:version.gt =< 3)endclassFirefox <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >BrowserendclassShape include Mongoid::Document field :x, type: Integer field :y, type: Integer embedded_in :canvasendclassCircle <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Shape field :radius, type: FloatendclassRectangle <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Shape field :width, type: Float field :height, type: Floatend

In the above example, Canvas, Browser and Firefox will all save in the canvases collection. An additional attribute _type is stored in order to make sure when loaded from the database the correct document is returned. This also holds true for the embedded documents Circle, Rectangle, and Shape.
Querying for Subclasses

Querying for subclasses is handled in the normal manner, and although the documents are all in the same collection, queries will only return documents of the correct type, similar to Single Table Inheritance in ActiveRecord.

# Returns Canvas documents and subclassesCanvas.where(name: "Paper")# Returns only Firefox documentsFirefox.where(name: "Window 1")

Associations

You can add any type of subclass to a has one or has many association, through either normal setting or through the build and create methods on the association:

firefox = Firefox.new# Builds a Shape objectfirefox.shapes.build({ x: 0, y: 0 })# Builds a Circle objectfirefox.shapes.build({ x: 0, y: 0 }, Circle)# Creates a Rectangle objectfirefox.shapes.create({ x: 0, y: 0 }, Rectangle)rect = Rectangle.new(width: 100, height: 200)firefox.shapes


