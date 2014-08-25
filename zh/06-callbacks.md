---
layout: post
title: 回调 
---
Callbacks

    Document Callbacks
    Relation Callbacks
    Observers 

Document Callbacks

Mongoid supports the following callbacks:

    after_initialize
    after_build
    before_validation
    after_validation
    before_create
    around_create
    after_create
    after_findSince 3.1.0
    before_update
    around_update
    after_update
    before_upsert
    around_upsert
    after_upsert
    before_save
    around_save
    after_save
    before_destroy
    around_destroy
    after_destroy 

Callbacks are available on any document, whether it is embedded within another document or not. Note that to be efficient, Mongoid only fires the callback of the document that the persistence action was executed on. This is that Mongoid aims to support large hierarchies and to handle optimized atomic updates callbacks can't be firing all over the document hierarchy.
	

Using callbacks for domain logic is a bad design practice, and can lead to unexpected errors that are hard to debug when callbacks in the chain halt execution. It is our recommendation to only use them for cross-cutting concerns, like queueing up background jobs.

classArticle include Mongoid::Document field :name, type: String field :body, type: String field :slug, type: String before_create :send_message after_save do |document| # Handle callback here.end protected defsend_message# Message sending code here.endend

Callbacks are coming from Active Support, so you can use the new syntax as well:

classArticle include Mongoid::Document field :name, type: String set_callback(:create, :before) do |document| # Message sending code here.endend

Relation Callbacks
Since 3.1.0

Mongoid has a set of callbacks that are specific to collection based relations - these are:

    after_add
    after_remove
    before_add
    before_remove 

Each time a document is added or removed from any of the following relations, the respective callbacks are fired: embeds_many, has_many, and has_and_belongs_to_many.

Relation Callbacks are specified as an option on the relation. The element added/removed is the parameter to the method you call via the callback. Example:

classPerson include Mongoid::Document has_many :posts, after_add: :send_email_to_subscribersenddefsend_email_to_subscribers(post) Notifications.new_post(post).deliverend

Relation Callbacks are not available to Observers.
Observers

Observer classes respond to life cycle callbacks to implement trigger-like behavior outside the original class. This is a great way to reduce the clutter that normally comes when the model class is burdened with functionality that doesn't pertain to the core responsibility of the class. Mongoid's observers work similar to ActiveRecord's. Example:

classArticleObserver <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Mongoid::Observerdefafter_save(article) Notifications.article("admin@do.com", "New article", article).deliver endend

Observers are available for any document, whether it is embedded within another document or not. Note that to be efficient, Mongoid only fires the observers of the document that the persistence action was executed on. This is that Mongoid aims to support large hierarchies and to handle optimized atomic updates callbacks can't be firing all over the document hierarchy.
Instantiation

Observers will not be invoked unless they are instantiated first. If you are using Rails, Mongoid will instantiate your observers automatically as long as you register them in your config/application.rb file like so:

config.mongoid.observers = :article_observer, :audit_observer

If you're not using Rails, then you will have to load and register your observers directly with Mongoid and afterwards instruct Mongoid to instantiate them before they will work. Instantiating an observer registers it with its observed model(s) so they will need to be loaded beforehand.

require "article_observer"require "audit_observer"Mongoid.observers = ArticleObserver, AuditObserverMongoid.instantiate_observers

Mapping

Observers will by default be mapped to the class with which they share a name. So CommentObserver will be tied to observing Comment, ProductManagerObserver to ProductManager, and so on. If you want to name your observer differently than the class you're interested in observing, you can use the Observer.observe class method which takes either the concrete class (Product) or a symbol for that class (:product). If an observer needs to watch more than one kind of object, this can be specified with multiple arguments.

classAuditObserver <  span style='font-size:12.0333px;font-style:normal;font-weight:400;font-family:Monaco,Menlo,Consolas,'   Courier Newmonospacecolorrgb  >Mongoid::Observer observe :account, :balancedefafter_update(record) AuditTrail.new(record, "UPDATED") endend



