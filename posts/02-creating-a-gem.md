## Creating Your Gem

Eureka! You've thought of a new idea for an open source library. Perhaps you already have some code tucked away in your application that you're extracting into a library, or perhaps you're starting from scratch. Either way, your next step is going to be setting up a gem folder to which you can then add code, tests, and documentation. As we saw previously, RubyGems are simply folders that follow certain patterns. You can build one from scratch quite easily, but luckily we have some tools that make it even easier.

### A Gem By Any Other Name...

Of course when you create your gem you're going to have to have something to call it. Naming may not seem very important as a first step, but the gem name ends up in folder names, file names, and Ruby code, so you usually want to come up with a name first. So how do you pick one?

Developers have created RubyGems with an incredibly wide range of names, from the straightforward to the absurd, from the clever to the downright crass. Depending on what you're building, however, there might be a convention to follow in your name. Here is an incomplete list of gem naming conventions:

* Your gem's name should be all lowercase with dash delimiters between words. Typically, the root module or class for your gem should be inferrable by class-casing your gem name. For example, if I wanted to call my library "Awesome Sauce" the gem name would be `awesome-sauce` and the root class/module would be `AwesomeSauce`.
* If you are building a gem that adapts an existing library (such as a C library for parsing JSON) the common pattern is simply to name your gem `libraryname-ruby` (e.g. `yajl-ruby` for the YAJL JSON library).
* If you are building a library that adapts another library for a framework (such as RSpec testing for Rails), the convention is to name it `libraryname-rails` (e.g. `rspec-rails`).
* Some libraries are somewhat like frameworks in their own right and allow for an open "constellation" of associated gems. These will usually suggest a naming pattern, but the common one is `frameworkname-libraryname` (for example, the Facebook strategy gem for the OmniAuth authentication framework is called `omniauth-facebook`).
* If you are building a Ruby library to access, for example, a specific service's web API, the gem is usually just named after the service (though in some cases follows the `service-ruby` pattern). You should be pretty confident about your long-term maintenance of a gem like this, as you will be claiming that gem name for all eternity. It would be unfortunate if the `twitter` gem fell into disrepair and was no longer a good way to access Twitter's API.

Ultimately a gem name is a lot like a brand name. Whether it's fair or not, a bad gem name can damage the popularity and success of a library just like a good one can make it more memorable and successful. When in doubt, try to include a keyword from your problem domain in the gem name (for example "auth" for an authentication library or "test" for a test framework). Avoid hard-to-spell words and elaborate metaphors (the `typhoeus` library has always been particularly hard to remember). Keep it short and simple, but ultimately, it's your library, and if you want to name it something funny or clever that's entirely your prerogative.

Before you settle on a name, you will need to make sure that it isn't already taken. The simplest way to do this is simply to visit `http://rubygems.org/gems/{YOUR_GEM_NAME}` in your browser. If you see a page that says "Page not found." then that gem is available.

### Tooling Up

Polishing Rubies is meant to be a guide that is both pragmatic and encourages best practices. To that end, whenever possible we will not be "doing it from scratch." When you're building an open source library you're building for multiple audiences: yourself, end users, and contributors. Using the right tools is important for all of the audiences, but is most important to encourage contribution.

When it comes to making contributors happy, you want to be using a toolchain that is as comfortable as possible to make it extremely easy for someone to check out your code and start working with it. We'll cover that a bit more later on.

Here a quick list of the tools that we'll be using throughout this guide to help us make a gem that follows community best practices:

* **[GitHub](https://github.com):** where you will be hosting the source code, issue reporting and wiki documentation for your gem
* **[Bundler](http://gembundler.com):** directory bootstrapping, dependency, and release management
* **[Rake](http://rake.rubyforge.org/):** task runner for tests, docs, and more
* **[Travis](http://travis-ci.org):** free, amazing continuous integration for open source projects
* **[Guard](https://github.com/guard/guard):** development process automation tool
* **[RSpec](https://github.com/rspec/rspec):** our test framework (my personal choice, but the built-in `Test::Unit` libraries included with Ruby are a perfectly acceptable alternative)
* **[Rubydoc.info](http://rubydoc.info):** automatic documentation generation for RubyGems. Always a handy place to link from your project's `README`
* **[Gemnasium](https://gemnasium.com/mbleigh):** tracks dependencies to make sure your library is always compatible with the latest versions of the libraries you use

We'll talk about the use of each of these tools as needed throughout the guide, but this can serve as a handy reference to the things that most (if not all) gems will eventually end up using. Right now we're only going to worry about using Bundler.

### Spinning Up a Gem

All right, now that all of the introductory material is out of the way, it's time to actually spin up your gem. This guide will be written assuming that you are using a POSIX-based shell system (such as Mac OS X or Linux) and that you have already installed Ruby 1.9.2 or later on your machine.

First make sure that you have Bundler installed on your system. Bundler contains a command-line helper called `bundle gem` that automatically generates a basic gem skeleton for us. To install Bundler you need to run this command in a terminal window:

```
gem install bundler
```

Once you've installed Bundler, you should change directories to the location that you want to create your gem folder (for me, that location is `~/code/gems`). Once there you will create the gem like so:

```
bundle gem my-gem
```

Where `my-gem` is your gem's name. Note that if you are building a command line tool you should add the `-b` flag to this command (`bundle gem my-gem -b`). You should see output that looks something like this:

```
      create  my-gem/Gemfile
      create  my-gem/Rakefile
      create  my-gem/.gitignore
      create  my-gem/my-gem.gemspec
      create  my-gem/lib/my-gem.rb
      create  my-gem/lib/my-gem/version.rb
Initializating git repo in /Users/mbleigh/code/gems/my-gem
```

Bundler has just generated a skeleton for your gem. You should recognize many of the files from our earlier discussion of the general structure of a gem. Some of the nice things that Bundler has done for us include:

* **.gitignore:** There are certain files that you will not want to have to your git repository. Bundler automatically excludes the most common of these.
* **version.rb:** Bundler creates `my-gem/lib/my-gem/version.rb` automatically. This file is used as a single place to track the current version of your library. By default, Bundler sets the version to `0.0.1`.
* **Rakefile:** The Rakefile created by Bundler is already set up with Bundler's gem helpers. These helpers allow you to release a new version of your gem (including creating a release tag in `git` and pushing the compiled gem to rubygems.org) with a single command.

Now that we've used Bundler to help us build our basic directory structure, we need to dig in and do a little housekeeping before we're ready to set up our development toolchain and start working.

### Gemspec: Metadata for Your Library

Bundler has automatically created the `gemspec` file that serves as a manifest of metadata about your gem. However, you will need to edit this file to add specific information before your gem will be properly set up. Your gemspec should look something like this:

```ruby
# -*- encoding: utf-8 -*-
require File.expand_path('../lib/my-gem/version', __FILE__)

Gem::Specification.new do |gem|
  gem.authors       = ["Michael Bleigh"]
  gem.email         = ["michael@intridea.com"]
  gem.description   = %q{TODO: Write a gem description}
  gem.summary       = %q{TODO: Write a gem summary}
  gem.homepage      = ""

  gem.executables   = `git ls-files -- bin/*`.split("\n").map{ |f| File.basename(f) }
  gem.files         = `git ls-files`.split("\n")
  gem.test_files    = `git ls-files -- {test,spec,features}/*`.split("\n")
  gem.name          = "my-gem"
  gem.require_paths = ["lib"]
  gem.version       = My::Gem::VERSION
end
```

Most of these fields are fairly straightforward. Authors lets you provide one or more names of the authors of the library, email is the contact email for each of the authors. The `executables`, `files`, and `test_files` fields each represent lists of files to include in the packaged gem. When a gem is built, only these files will be included. By default Bundler uses some handy command-line magic with `git` to programmatically determine the file lists. For most circumstances, you shouldn't need to change these lines.

At this point you need to make a few changes to this file to follow best practices:

1. **gem.description:** you should write a single-sentence description of your gem in this field. Note that `%q{}` is an alternative way to declare a string in Ruby. Bundler uses this by default so that you can use double and single quotes within the description without needing to escape them. Your maximum length should be no more than around 140 characters (like a tweet!).
2. **gem.summary:** you can just copy and paste the gem description into this field for now.
3. **gem.homepage:** this should generally be set to the location of your repo on GitHub unless you have created a specific website for your library.

The last step of your initial setup is to ensure that the constant and folder names generated by Bundler matches the constant and folder names you want to use for your library. Sometimes Bundler gets this right, and sometimes it doesn't. For example, in the above `gemspec` the referenced version is `My::Gem::VERSION` when I would actually want it to be `MyGem::VERSION`. To change this, I need to modify the constant in three places:

1. The version reference in the `gemspec`
2. The `lib/my-gem.rb` file
3. The `lib/my-gem/version.rb` file

Unless your library is going to be very small, you will usually want your base constant (for example `MyGem`) to be a **module**, not a **class**. This allows you to easily namespace all of the different classes and components of your gem in a simple and sane way.

### Underscores and Dashes

RubyGems have an unfortunate conflict of common patterns: gem names are almost always delimited with dashes while folder and file names are almost always delimited with underscores. This is partially to make for inferrable mappings from file names to class names (a pattern that is actively utilized by Rails to automatically load constants based on their file names). So if I have a constant called `MyGem::VERSION` it should be located in `lib/my_gem/version.rb` even though my gem's *name* is `my-gem`.

When Bundler generates your gem (if you have dashes in the name) it creates a folder inside the `lib` directory of the same name. I would recommend modifying this to use underscores in place of dashes. To do this, you will need to do the following:

1. Rename the directory (e.g. `mv lib/my-gem lib/my_gem`).
2. Alter the first line `require` in `lib/my-gem.rb` to use the underscored directory name.
3. Alter the `require` in the `gemspec` file to use the underscored directory name.

If all of this weren't confusing enough, there is **another** pattern in Ruby that the only file that should be added to the root load path (the first level of your gem's `lib`) should be named identically to the gem. So `lib/my-gem.rb` is the **one and only place** that you should allow dashes in your filenames.

Following such needlessly complicated naming schemes may feel like a waste of time, but it most certainly isn't. By following the community conventions you:

1. Make your library easier to "guess" how to use because developers can use common idioms to which they have become accustomed.
2. Make your library auto-requireable by Bundler (which looks for the `my-gem.rb` file and requires it automatically in frameworks such as Rails)
3. Make your library easier for contributors to join by providing familiar patterns and locations for code

### Library Dependencies

As with a man, no gem is an island (well, ok, very few gems are islands). Nearly all gems are going to have **dependencies** upon other libraries that they require in order to function. These dependencies are explicitly declared in the `gemspec` so that when a gem is installed, all of the gems that are needed to run that gem can automatically be installed with it.

There are two types of gem dependencies, **runtime dependencies** and **development dependencies**. Runtime dependencies are libraries that your gem needs in order to function. For example, if you built a Rails extension, you would need to add Rails to the runtime dependencies of your project. Development dependencies, on the other hand, are gems that are only needed by people who are going to contribute to the code of the gem. Gems are, by default, installed only with the runtime dependencies. This avoids cluttering up an environment with development dependencies if they aren't needed.

Adding dependencies to your `gemspec` is extremely simple; all you need to do is add a line to the file for each dependency:

```ruby
# -*- encoding: utf-8 -*-
require File.expand_path('../lib/my-gem/version', __FILE__)

Gem::Specification.new do |gem|
  # ...

  gem.add_dependency 'rails'
  gem.add_development_dependency 'rspec', '~> 2.7'
end
```

In addition to specifying the gem name of a dependency, you can also specify a version of the gem to get even more specific. Versioning is an important topic and one that we will cover later. For now, it's OK if you just add your dependencies without specifying the version. RubyGems will default to the latest version if no specific version is required.

Now you've created the basic structure of your gem. You understand the `gemspec` manifest file, a little bit about the structure of the app, and the somewhat-confusing-but-it-makes-sense-eventually-trust-me file naming conventions for Ruby libraries. You've also hopefully added the gem dependencies that you already know about (don't worry, you can always add more later). There's only one more thing to do: commit your code! It's great to have your first commit at this point because you haven't actually added any specific code and this serves as a baseline. So `git add .` and `git commit -m "Initial Import"` and you're on your way to building a first-class Ruby library!

Next time, we'll get our new gem set up with all of the bells and whistles you need to make open source development hum.