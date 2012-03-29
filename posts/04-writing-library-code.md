# Writing Library Code

Can you believe it? It's actually time to start writing the code for your gem! Now, in this part of the guide you'll be more "on your own" than up to this point. I don't know what kind of open source library you're writing, whether it's an extension to an existing library, a simple utility, or a complex, sprawling project that will change the face of development forever. What I do know, however, is that there are some common things that you may want to do that have community best practices attached.

### File Structure, Naming, and Requirement

While it may not *seem* vital, how you name your files and where you put them can be a critical component of your library's ease of use for developers. This is particularly true because on a GitHub repository page one can simply press `t` and start typing a filename, then look at the code for that file by pressing return. This is the way that many people will try to take a quick peek at the internals of your gem, so by creating sane, *easily guessable* filenames you are making it simple for others to read your code.

In general, you will have one class per file and one folder per module namespace, so for instance:

* **MyGem** becomes `lib/my-gem.rb` (this is a special case, see [part 2](http://intridea.com/blog/2012/3/15/polishing-rubies-part-ii))
* **MyGem::Widget** becomes `lib/my_gem/widget.rb`
* **MyGem::Widgets::FooBar** becomes `lib/my_gem/widgets/foo_bar.rb`

There are, however, a few corner cases in which each class may not have its own file. One is a private class that is meant to be used only internally and only in the scope of another class. Another are the exception classes for your library. We will cover exception classes in the next section.

When a developer wants to use your library, they should be able to do so (in almost all cases) by making a single `require` statement that is identical to the gem name. That means that in the root file you need to make any additional `require` statements necessary for your gem to function. So if I have a `MyGem` module, a `MyGem::Widget` class, and a `MyGem::Widgets::FooBar` class, the `lib/my-gem.rb` file in my gem might look like this:

```ruby
require 'external_library' # require any external dependencies

module MyGem # it is best to declare the base module at the top
end

require 'my_gem/version' # created by Bundler
require 'my_gem/widget'
require 'my_gem/widgets/foo_bar'
```

By requiring all of the files necessary for your gem to run in the base file you make it easier for developers to use your library. Some gems, however, may be made up of multiple parts that could be used independently of each other. ActiveSupport, for example, provides a large number of useful utilities that, while they function together, can also function separately.

If I add `require 'active_support'` to my code I load *all* of ActiveSupport. While this may be what I want in some cases (like inside a Rails application) in other cases I may just want a specific piece of ActiveSupport. Luckily, ActiveSupport is designed to handle this well. If I, for instance, add `require 'active_support/core_ext'` I will only be loading the Ruby core extensions that are a part of ActiveSupport.

How can you make this work in your library? It's quite simple: your base file should, when required, require all the other parts of your library. However, each part of your library should, at its top, require any other parts or external dependencies so that it may be included *without* the user having previously required the base file. Let's take a look at an example:

```ruby
# in lib/my-gem.rb

require 'my_gem/widget'
require 'my_gem/widgets/foo_bar'
reuqire 'my_gem/widgets/baz'

# in lib/my_gem/widget.rb

require 'external_library'

module MyGem
  class Widget
  end
end

# in lib/my_gem/widgets/foo_bar

require 'my_gem/widget'

module MyGem
  module Widgets
    class FooBar < MyGem::Widget
    end
  end
end
```

Each of the files in the above example can be required independently, giving developers the flexibility to use only a subset of your library's functionality if needed. Remember that `require` statements will only load the code from a file once, so it is safe to `require` the same file multiple times.

### Exception Classes

As a library author you are creating a black box: you tell developers how to send messages to the box, the box sends messages back, but a well-designed library does not expose internal implementation details to its end users. In service of creating a properly bounded library, your application should only raise errors that you allow it to raise.

Sometimes you will be able to use existing error classes. For instance, if you might raise an `ArgumentError` if invalid parameters are passed to a method in your library or a `NotImplementedError` if you are building some kind of abstract interface. Many times, however, you will want to create one or more custom exception classes to provide insight into what happened when things go wrong.

Exception classes usually end up in one of two places in a library: either in the base file (`lib/my-gem.rb`) or, if there are more than one or two error classes, all together in a file such as `lib/my_gem/errors.rb`. It is entirely possible to create a custom exception class in a single line:

```ruby
module MyGem
  class Error < StandardError; end
end
```

This allows you to raise out an error that can be caught by users of your library when exceptions occur:

```
raise MyGem::Error, "This is the error message."
```

If you wish to create more than one custom exception class, it is a good practice to have a single base `MyGem::Error` class and several subclasses underneath it. This way, users of your library can rescue from any exception it might generate simply with `rescue MyGem::Error`.

One other thing you might want to do is store some additional information along with an error; for instance, in an API client library if you receive a non-success status code you may want to include the HTTP response so that more information can be obtained:

```ruby
module MyGem
  class Error < StandardError; end
  class ResponseError < MyGem::Error
    attr_reader :response

    def initialize(response, message)
      @response = response
      super(message)
    end
  end
end

# Elsewhere...

raise MyGem::ResponseError.new(response, "Invalid response from HTTP client.")
```

Because you are providing library code you will usually want to wrap any internal exceptional scenarios with your own exception class. For instance, if there is a method that you call from another library that will sometimes raise exceptions, you should rescue from those exception classes and wrap it into your own:

```ruby
module MyGem
  class SomeClass
    def some_method
      ExternalLibrary.dangerous_method_call
    rescue ExternalLibrary::Error => e
      raise MyGem::Error, "Some dangerous method call didn't work."
    end
  end
end
```

Proper exception handling makes your library well-encapsulated and easy to use and also helps your users to report issues when they occur.

### Library Configuration

Oftentimes in a library there is various "root-level" configuration that may need to be performed. There are lots of different ways to handle such configuration (including a number of gems that you can depend upon to do the heavy-lifting) but, with a little bit of elbow grease, you can write your own configuration class that is robust without being cumbersome.

To begin, let's create a new file `lib/my_gem/configuration.rb` that will contain our configuration class. Now let's fill it up:

```ruby
module MyGem
  class Configuration
    attr_accessor :option1, :option2

    def initialize(config = {})
      config.each_pair do |key, value|
        self.send("#{key}=", value)
      end
    end
  end
end
```

This gets us started with two configuration options (`:option1` and `:option2`) and can be used like so:

```ruby
config = MyGem::Configuration.new(option1: 'foo', option2: 'bar')
config.option1 # => 'foo'
config.option1 = 'baz'
config.option1 # => 'baz'
```

As you can see, this gives us the beginnings of what we might want from library configuration: there are a few different options, we can read and write each option individually and we can also initialize their values with a hash. So what might we want next? Well, it's usually best to explicitly define default options we might have:

```ruby
module MyGem
  class Configuration
    DEFAULT_CONFIGURATION = {
      option1: 'foo', 
      option2: nil
    }

    attr_accessor :option1, :option2

    def initialize(attributes = {})
      DEFAULT_CONFIGURATION.merge(attributes).each_pair do |attribute, value|
        self.send("#{attribute}=", value)
      end
    end
  end
end
```

That was easy enough: our default options are set as a *constant* beneath our Configuration class, and we then merge in the options that are passed to the constructor to bootstrap our configuration. Note also that in this example `:option2` is defaulted to `nil`. Each option should be given an *explicit* default value so that developers who are reading your code know all of the options that are available.

Next up we will make our Configuration class friendly to people who want to use block-style declarative configuration:

```ruby
module MyGem
  class Configuration
    DEFAULT_CONFIGURATION = {
      option1: 'foo', 
      option2: nil
    }

    attr_accessor :option1, :option2

    def initialize(attributes = {}, &block)
      configure(DEFAULT_CONFIGURATION.merge(attributes), &block)
    end

    def configure(attributes = {})
      attributes.each_pair do |attribute, value|
        self.send("#{attribute}=", value)
      end

      yield self if block_given?

      self
    end
  end
end
```

As you can see we've added a `#configure` method that takes a hash and sets configuration values or takes a block and yields itself. We also added block-yielding capabilities to the constructor. This allows us to set configuration like so:

```ruby
c = MyGem::Configuration.new do |config|
  config.option1 = 'baz'
end

c.option1 # => 'baz'

c.configure do |config|
  config.option1 = 'wonk'
end

c.option1 # => 'wonk'
```

Look familiar? Many gems utilize this block-yielding method of configuration; it's a community pattern and one that is nice and easy to implement. Now that we've built a simple configuration class, we need to hook it into our root module. So let's open up `lib/my_gem.rb` and add the following:

```ruby
module MyGem
  def self.config
    @config ||= Configuration.new
  end

  def self.configure(attributes = {}, &block)
    config.configure attributes, &block
  end
end
```

By *memoizing* an instance of the configuration class at `MyGem.config`, we have a single persistent configuration object for our gem. We can now configure our library exactly like many others:

```ruby
MyGem.configure do |config|
  config.option1 = 'bar'
end

MyGem.config.option1 # => 'bar'
```

Because we support both block-yielding and hashes, we also make it easy for people to use, for instance, a YAML configuration file and pass the relevant configuration in that way.

### Keep it Clean

The most important aspect of writing library code is just to write the best code you can write! Every best practice and code style that you've heard of is doubly important when you're writing code meant to be read and used by an entire community of developers. That being said, no library ever ships with perfect code. It's often better to do your best and get *something* out the door than to hold off and hold off while you tweak things ad infinitum. You can always fix it in the next version!