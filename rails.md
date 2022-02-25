# Rails

## Contents

- [Autoloading](#autoloading)
- [Console](#console)

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


