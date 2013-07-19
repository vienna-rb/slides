#Faster Tests in Rails
#Presenter Notes

A short introduction

Comprehensive, not all-integrating

partially blasphemic

---
# Anton Bangratz

- anton.bangratz@radarservices.com
- https://abangratz.github.io/
- https://github.com/abangratz
- https://twitter.com/tony\_xpro

---
#A Common Problem: Rails Tests/Specs are slow
#Presenter Notes

GREAT! You have a lot of tests!

you are not alone

you might be testing too much

you should check if you are testing the right things

---
#1. Upgrade to Ruby 2.0
#Presenter Notes

 - Faster Startup
 - Better compatibility with Rails 4
---
# After Upgrade:
  - Startup is faster
  - no need for Spork et al
  - use of autotest, guard etc is definitely viable without the old tricks

##Example:

Autotest, all tests:

    Finished in 0.6798 seconds
    124 examples, 0 failures, 4 pending

Autotest, one file:

	Finished in 0.22468 seconds
	8 examples, 0 failures


---
#2. Modularize Horizontally
---
#Different Techniques

##Vertically

    |   Database   |     Slave     |
	+--------------+---------------+
	| App Server 1 | App Server 2  |
	+--------------+---------------+
	|       Load Balancer(s)       |

Great for production. Doesn't help in Tests. Good for scaling, so don't dismiss it.

##Horizontally

    |        ORM         |
    |        API         | < Use Workers for 'invisible' Tasks
    |        DAO         |
    |  Business Objects  | <
    |     Presenters     | < Via Adapters
    |  View  Generators  | <

Good for testing. Doesn't matter in Production. Makes changes even easier.

#Presenter Notes
SOC (and SOLID principles),

Splitting of Full Stack Tests (Rendering vs Data Manipulation),

Use full stack tests for critical paths in the API only,

Use e.g. Sham::Rack for providing API endpoints for unit and stack tests

---
#Unit Tests
---
#Use Plain Old Ruby Objects (PORO)
---
#Why not use mock_model?

Consider: you are including all of this:

    !ruby
	#...
	stubs = stubs.reverse_merge(:id => next_id)
    stubs = stubs.reverse_merge(:persisted? => !!stubs[:id],
                                :destroyed? => false,
                                :marked_for_destruction? => false,
                                :valid? => true,
                                :blank? => false)

    double("#{model_class.name}_#{stubs[:id]}", stubs).tap do |m|
      m.singleton_class.class_eval do
      include ActiveModelInstanceMethods
      include ActiveRecordInstanceMethods if defined?(ActiveRecord)
      include ActiveModel::Conversion
      include ActiveModel::Validations
    end
    if defined?(ActiveRecord)
      [:save, :update_attributes, :update].each do |key|
        if stubs[key] == false
          m.errors.stub(:empty? => false)
        end
      end
    end
    m.__send__(:__mock_proxy).instance_eval(<<-CODE, __FILE__, __LINE__)
	#...
## It's not bad per se, but if you just use state of the models, don't use it
---
#Solution 1: Use an ad-hoc class

	!ruby
	class MyTestDummyModel < OpenStruct
	  extend ActiveModel::Naming
	end

	describe TestController do
	  context "GET /show" do
	    let (:object) { MyTestDummyModel.new(id: index) }
		it "should work" do
		  ObjectModel.stub!(:find).with("5").and_return(object)
		  get :index, id: 5
		  response.should be_success
		end
	  end
	end

---
#Solution 2: Dependency Injection

## A. Controller

	!ruby
	class TestController < ActionController::Base
	  attr_accessor :single_fetch_function
	  def initialize
		super
		@single_fetch_function ||= ObjectModel.method(:find)
	  end

	  def show
		@object =self.single_fetch_function.(params[:id])
	  end

	end

## B. Test

	!ruby
	describe TestController do
	  context "GET /show" do
		it "should work" do
		  controller.single_fetch_function =
		    ->(x) { MyTestDummyModel.new(id: x) }
		  get :index, id: 5
		  response.should be_success
		end
	  end
	end
### You can use a mock here, but only if you need the interaction

---
#WARNING: Stop Testing Tested Code
#Presenter Notes
- Libraries, ORMs

---
#Except When You Changed Something
#Presenter Notes
- and then test only that something

---
#Golden Rule: Don't Test Anything You Didn't Implement
#Presenter Notes
- Which leads me to:

---
#Integration Tests
---
#Horizontal Split

##Disadvantages:

- Full stack tests have to be in sync
- Test more than 1 app
- &hellip; now, what to do?

---
#Treat Every Part As 3rd Party Code
---
#How?

- Test at the interfaces (if both sides are tested rigorously, there's nothing to worry)
- Mock those interfaces (you know what to send and what to expect, don't use the real app)
- Don't trust yourself.
- I mean, REALLY, DON'T.
#Presenter Notes
- Sham::Rack
- &hellip;

---
#How to Avoid Regressions
- Test First
- API Versioning
- Rigorously hunt down bugs and write tests for them

---
#That's it, folks
---
#Oh, BTW
---
#We are hiring!
  - (Ruby/Rails) Developers
  - DevOps

Talk to me after the talks ;)


<img src="radar_services.png" />
---
