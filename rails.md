# Rails

## Contents

- [Autoloading](#autoloading)
- [Console](#console)
- [Routing])(#routing)
- [Generators](#generators)

## Autoloading

Generally in Ruby applications, you have to `require` the dependencies of a file. In Rails, autoloading is used so that this is not necessary. Instead, Rails will look for an appropriate class based on naming conventions. 

```ruby
# app/controllers/health_controller.rb

class HealthController < ApplicationController
  def index
    render :plain => "Everything is okay at %s" % TimeHelpers::Format.now_formatted
  end
end

# app/helpers/time_helpers/format.rb
module TimeHelpers
  module Format
    def now_formatted
      time = Time.now
      "%d:%d:%d" % [time.hour, time.min, time.sec]
    end
  end
end
```

## Console

As well as running up the application, it is also possible to run a Rails console. This allows for various operations that can be useful for exploring, testing and debugging a project including:

### Interact with models and the db

```
> User                          # Take a look at the User model
> User.methods                  # See what methods are available on the User model
> User.all / User.find(...)     # Query the db
```

### Send requests to the application

```
> app.get "/health"
=> 200 
```

### Interact with other objects from the application

```
> helper.now_formatted # Use the `helper` object to call helper methods
```

## Routing

Routing in Rails apps is all defined in the `config/routes.rb` file. The snippet below contains examples of a number of common patterns. More information can be found in the [Rails docs](https://guides.rubyonrails.org/routing.html).

```ruby
# config/routes.rb

Rails.application.routes.draw do
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html
  
  root "articles#index" # Define where the root path (i.e /) goes.

  get '/search', to: 'search#index' # Define where a particular method and route go.

  get '/about-us', to: 'about#index' # GET /about-us will be routed to the `index` method of the `AboutController` (expected in app/controllers/about_controller.rb)

  get '/posts/:id' to: 'posts#show' # Route matching requests (e.g. GET /posts/17) to posts#show with the id param set.

  resources :users # This is a helper method which provides a series of routes pointing to the UsersController covering all of the CRUD methods.
  
  # This is a similar convenience method which doesn't rely on sending an id for individiual items.
  # This might be because there is only of them or because there is a different way of determining the individual item of interest.
  # In this case, for example, the relevant user would be the one who is currently logged in.
  # Requests are still routed to the pluralised controller (e.g. UsersController) and this can be used in conjuctions with the resources method.
  resource :user

  # Blocks can be used to nest routes
  # In this case, the various comment routes would be after `/articles/:article_id`
  # e.g. GET /articles/:article_id/comments/:comment_id would show a particular comment on a particular article
  resources :articles do
    resources :comments
  end

  # Namespaces can be used to nest routes, this will also expect the controllers to be nested in
  # an equivalent directory
  # e.g. GET /admin/keys would go to Admin::KeysController in app/controllers/admin
  namespace :admin do
    resources :keys
  end

  # Scope can be used to send routes to the nest controllers without affecting the URL
  # e.g. GET /keys would go to Admin::KeysController in app/controllers/admin
  scope module: 'admin' do
    resources :keys
  end

  # Or it can be used to route to the KeysController with the nested URL
  # e.g. GET /admin/keys would go to KeysController in app/controllers
  scope '/admin' do
    resources :key
  end

  # Only can be used to limit the routes that the resources method created
  resources :users, only: [:index, :new, :create]

  # Mount can be used to route requests to another Rack app instead of a rails controller
  mount MetricsApp, at: '/metrics'

  # Procs can be used to define really simple endpoint directly in the routes file
  get '/status', to: proc { [200, {}, ['OK']] }

end
```

## Generators

Generators can be used to create the boiler plate for a number of common Rails items. 

Running `rails generate` will list the items can be generated. 

The most common items to generate are models and controllers. Running these generators will create not only the items but also the various related items. For example, generating a controller will also add an entry to the `config/routes.rb` file. 

There is also a `scaffold` generator which will create a full range of services including a model, database migration, controller, views, and tests. 

It is also possible to [define your own generators](https://guides.rubyonrails.org/generators.html) to boilerplate custom elements.

The [Rails docs](https://guides.rubyonrails.org/command_line.html#bin-rails-generate) contain more information on the available generators and their options. 

