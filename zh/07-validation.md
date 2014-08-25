---
layout: post
title: 验证
---

Validations

Mongoid includes ActiveModel::Validations to supply the basic validation plus an additional associated and uniqueness validator.

See ActiveModel::Validations documentation for more information.
	Mongoid behaves slightly different to Active Record when using #valid? on already persisted data. Active Record's #valid? will run all validations whereas Mongoid's #valid? will only run validations on documents that are in memory as an optimization.

Common options that can be passed to all validations:

    :allow_nil Specify whether to validate on a nil attribute.
    :if Only run if the supplied value evaluates to true.
    :on Only run when specified, supports :create and :update.
    :unless Only run if the supplied value evaluates to false.

In addition to those validations, information is provided with each macro about its specific options.
14072009.11828
Validation 	Macro	Options

Validate acceptance of terms.
	

validates_acceptance_of :termsvalidates :terms, acceptance: true

	

:message:accept# Specify the accepted value. Default: 1.

Validate associated documents when the parent is validated. This only validates documents in memory.
	

validates_associated :albumsvalidates :albums, associated: true

	

:message

Validate confirmation of a field, with a "_confirmation" suffix.
	

validates_confirmation_of :passwordvalidates :password, confirmation: true

	

:message

Validate items are not in a list.
	

validates_exclusion_of :employers, in: [ "SoundCloud" ]validates :employers, exclusion: { in: [ "SoundCloud" ] }

	

:in# Required list of values.:message

Validate the format of a field.
	

validates_format_of :title, with: /\A\w+\Z/validates :title, format: { with: /\A\w+\Z/ }

	

:allow_blank# Whether the field can be blank.:in# A list or range of values.:message:with# A regular expression to match.:without# A regular expression not to match.

Validate items are included in a list.
	

validates_inclusion_of :employers, in: [ "SoundCloud" ]validates :employers, inclusion: { in: [ "SoundCloud" ] }

	

:allow_blank# Whether the field can be blank.:in# Required list of values.:message

Validate the length of a field.
	

validates_length_of :password, minimum: 8, maximum: 16validates :password, length: { minimum: 8, maximum: 16 }

	

:allow_blank# Whether to validate blank attributes.:in# The range the length can fall within.:maximum# The maximum length of the attribute.:message:minimum# The minimum length of the attribute.:tokenizer# A block tokenizer.:too_long# Custom message if too long.:too_short# Custom message if too short.:within# Range the length can fall within.:wrong_length# Custom message for an incorrect length.

Validate the numericality of a field.
	

validates_numericality_of :age, even: truevalidates :age, numericality: { even: true }

	

:equal_to# A value the field must be exactly.:even# Set that the value must be even.:greater_than:greater_than_or_equal_to:less_than:less_than_or_equal_to:message:odd# Set that the value must be odd.:only_integer# Set whether the value has to be an integer.

Validate that an attribute exists. Note that if you add a presence validation to a relation, then Mongoid will enable autosave for that relation.
	

validates_presence_of :namevalidates :name, presence: true

	

:message

Validate that an attribute is unique. Note that for embedded documents, this will only check that the field is unique within the context of the parent document, not the entire database.
	

validates_uniqueness_of :namevalidates :name, uniqueness: true

	

:message:case_sensitive# Whether to use case sensitive matching.:scope# Scope checks to the value of this field.


