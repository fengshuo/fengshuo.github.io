---
title: Serializing and JSON API resources in Rails
categories:
- DevEnv
- Shell
tags:
- rails
- ruby
- serializer
- active model serializer
- json
- api
- backend
- JSONAPI::Resources
description: How to use active model serializer and jsonapi::resources in rails to
  create API
date: 2019-02-06
author_profile: true
classes: wide
---

# Serializing, resources in rails

The goal of this article is to understand how to use `active_model_serializers` and `JSONAPI::Resources` and the reason why we should use them.

## Create an api only rails app

What I want to achieve is to use rails as backend talks to frontend via api, that means the rails app should be api only, and since rails already have `render :json` out of the box, let's get the first version of the app working: (I use postgres you don't have to keep the postgres related part)

```bash
rails new my-app --api --database=postgresql
```

You will find the app code generate with this setup is lighter, some directories you see won't be there anymore, such as `app/assets/`, `lib/assets/`, `app/views/layouts/application.hmtl.erb` and some gems related to the view part are not installed, such as `sass-rails`, `coffee-rails`, the `app/controller/application_controller.rb` looks a little different too, since it is now inheriting from `ActionController::API` instead of `ActionController::Base`,also the line `protect_from_forgery with: :exception` is gone too. (see a more comprehensive list [here](https://hashrocket.com/blog/posts/how-to-make-rails-5-api-only))

Since the backend and frontend are separate, we will need to enable CORS:

In the Gemfile, uncomment `gem 'rack-cors'` , create a new file `config/initializers/cors.rb`, it should like this:

```ruby
Rails.application.config.middleware.insert_before 0, "Rack::Cors" do
  allow do
    origins '*'
    resource '*',
      headers: :any,
      methods: %i(get post put patch delete options head)
  end
end
```

Now let's create some models as a simple example:

```bash
# one user has many pets
rails g User name:string age:integer
rails g Pet name:string fav_food:string user:references
# or rails g Pet name:string fav_food:string user_id:integer
```

update the model files

```ruby
class User < ApplicationRecord
	has_many :pets
end

class Pet < ApplicationRecord
  belongs_to :user
end
```

generate the controller without view assets (note the plural)

```bash
rails g controller Users --skip-assets
rails g controller Pets --skip-assets
```

Then modify the controller, for this simple example we will just use GET request, update the router:

```ruby
Rails.application.routes.draw do
  resources :pets, only: [:index, :show]
  resources :users, only: [:index, :show]
end

# pets_controller
class PetsController < ApplicationController
  def index
    render json: Pet.all
  end

  def show
    render json: Pet.find(params[:id])
  end
end

# users_controller
class UsersController < ApplicationController
  def index
    render json: User.all
  end

  def show
    render json: User.find(params[:id])
  end
end
```

Let's create some dummy data in rails console after `rake db:migrate:`

```bash
a = Person.create({name: "aaa", age: 23})
b = Person.create({name: "bbb", age: 25})
a.cats.create({name: "Fluffles", fav_food: "friskies"})
a.cats.create({name: "Spot", fav_food: "salmon"})
a.cats.create({name: "Furtha",fav_food: "chicken"})
b.cats.create({name: "Meowserino", fav_food: "Garbanzo Beans"})
b.cats.create({name: "Boomer", fav_food: "beef"})
b.cats.create({name: "Mr. Whiskers", fav_food: "Gouda Cheese"})
```

Now run the rails server and make request to `/users` or `/pets` api, you should see all attributes are included in the response, including `create_at` and `update_at`, But we want more control over the response, for instance, we don't need to show those timestamp to front end.

## [active model serializers](https://github.com/rails-api/active_model_serializers/tree/v0.10.6/docs)

To get better control of the response, we will install `gem 'active_model_serializers'` then use the generator: `rails g serializer pet` this will generate a new directory `app/serializer`

```ruby
class PetSerializer < ActiveModel::Serializer
  attributes :id, :name, :fav_food
  # the attributes here are the whitelist of the response
end

# add relationship
class PetSerializer < ActiveModel::Serializer
  attributes :id, :name, :fav_food, :user
  # it will display all user info as well, which is redundant
end
```

But usually we only need the id of the user in the pet response, we can enhance it with custom functions like this:

```ruby
class CatSerializer < ActiveModel::Serializer
  attributes :id, :name, :owner
  def owner
    {owner_id: self.object.person.id}
  end
end
```

There are more ways to add control, such as to [make an attribute conditional](https://github.com/rails-api/active_model_serializers/blob/v0.10.6/docs/general/serializers.md#attribute):

```ruby
attribute :private_data, if: :is_current_user?
attribute :another_private_data, if: -> { scope.admin? }

def is_current_user?
  object.id == current_user.id
end
```

You can also use a [different adapters](https://github.com/rails-api/active_model_serializers/blob/v0.10.6/docs/general/adapters.md) such as JSON api.

## [JSONAPI::Resources](http://jsonapi-resources.com/v0.9/guide/basic_usage.html)

You might be thinking active model serializer seems to be good enough, but if you consider the [JSON API spec](https://jsonapi.org/format/), active model serializer(AMS) might not be enough, [according to the creator](http://www.cerebris.com/blog/2014/08/22/introducing-jsonapi-resources/) of the JSONAPI::Resources, even though AMS has support for JSON API spec with the adapter, its focus is serializers not resources.

> The primary reason we developed JR is that AMS is focused on serializers and not resources. While serializers are just concerned with the representation of a model, resources can also act as a proxy to a backing model. In this way, JR can assist with fetching and modifying resources, and can therefore handle all aspects of JSON API.

let's see how it works after adding `gem 'jsonapi-resources'` and `bundle`:

first include the module in controller, it could be in the `ApplicationController` or under namespace such as:

```ruby
module Web
  class SupportsController < ApplicationController
    include JSONAPI::ActsAsResourceController
  end
end
```

some config update in `config/environments/development.rb`

```ruby
config.eager_load = true
config.consider_all_requests_local       = false
```

Create models and relationships similar to the pet and user example.

Create controllers:

```bash
rails g controller Contacts --skip-assets
rails g controller PhoneNumbers --skip-assets
```

Now create `app/resources` directory, make resource file for each model in a standard way such as `phone_number_resource.rb`, `user_resource.rb`

```ruby
class ContactResource < JSONAPI::Resource
  attributes :name_first, :name_last, :email, :twitter
  has_many :phone_numbers
end

class PhoneNumberResource < JSONAPI::Resource
  attributes :name, :phone_number
  has_one :contact

  filter :contact
end
```

The add this in routes.rb:

```ruby
jsonapi_resources :contacts
jsonapi_resources :phone_numbers
```

Now you can create new contact or phone numbers with POST request, then make GET request you should see something like this:

```js
{
    "data": [
        {
            "id": "1",
            "type": "phone-numbers",
            "links": {
                "self": "http://localhost:3000/phone-numbers/1"
            },
            "attributes": {
                "name": "home",
                "phone-number": "(603) 555-1212"
            },
            "relationships": {
                "contact": {
                    "links": {
                        "self": "http://localhost:3000/phone-numbers/1/relationships/contact",
                        "related": "http://localhost:3000/phone-numbers/1/contact"
                    }
                }
            }
        },
				......
    ]
}
```

And because we have filter in phone number resources, an API request like this `[localhost:7070/phone-numbers?filter[contact]=1](http://localhost:7070/phone-numbers?filter[contact]=1)` would return phone numbers belongs to contact id=1.

You can also get context that is available in controller, for instance:

```ruby
class ApplicationController < JSONAPI::ResourceController
  def context
    {current_user: current_user}
  end
end
```

And you can specify get the underlying model with `@model`. You can also specify which attribute is fetchable or updatable, set up filter, pagination, and custom links.
