---
layout:             post
title:              Rails, Rspec, and testing controllers with the same name
date:               2014-09-11 16:53:39 -0400
tags:               
---

It's important to write tests for your code.

However, I was trying to write some tests for a pair of rails controllers with the same name, but different modules. The controllers look something like this:

```rb
# app/controllers/my_controller.rb
class MyController < ApplicationController
...
end

# app/controllers/subsection/my_controller.rb
class Subsection::MyController < Subsection::ApplicationController
...
end
```

Then I wrote Rspec tests for each, which all work when run one file at a time:

```rb
# spec/controllers/my_controller_spec.rb
require 'spec_helper'
describe MyController do
...
end

# spec/controllers/subsection/my_controller_spec.rb
require 'spec_helper'
describe Subsection::MyController do
...
end
```

However, when I ran all the tests at once using '$ rake spec', the tests for Subsection::MyController
always tried to use the MyController at the base level for the controller, which led to an error like this:

```
ActionController::UrlGenerationError:
       No route matches {:action=>"the_action", :controller=>"my_controller", :id=>"123"}
```

I finally found the solution after chasing the wrong symptoms. I can't explain it better than [this guy](http://stackoverflow.com/a/19028366/251543). In short, require the class you're testing at the top of your test files to ensure the right class is being loaded:

```rb
# spec/controllers/my_controller_spec.rb
require 'my_controller'
require 'spec_helper'

describe MyController do
...
end

# spec/controllers/subsection/my_controller_spec.rb
require 'subsection/my_controller'
require 'spec_helper'
describe Subsection::MyController do
...
end
```
