# The What and Why of Gems

Building an open source library can be a daunting task if you've
never done it before. How should I structure the project? What do I need
to include in terms of documentation? What tools can I use to make my
library friendly for others to contribute?

We'll get to all of that. But before we do, I think it's worthwhile to
examine the RubyGems system, how it works, and what exactly makes up a
gem. Once you understand all of that, the idea of creating a gem will be
neither confusing nor mysterious but rather just another tool in your
arsenal to be used to help make your projects cleaner, more modular, as
well as to help you contribute to the open source community.

This post series assumes that you have already installed Ruby on your
development machine. You can visit the [Ruby language download 
page](http://www.ruby-lang.org/en/downloads/) if you need help getting
started with Ruby.

### RubyGems: A Package Management System for Ruby

All programming languages have support for **libraries**. Libraries are
collections of reusable code that are usually grouped around a common
purpose. There can be libraries for just about anything, from advanced
processing of strings to higher level math functions to entire
encapsulated applications.

While all programming languages have libraries, *not* all programming
languages have a system to deliver those libraries as simple and elegant
as RubyGems. RubyGems is an internet-aware package management system
that allows for the download and installation of new libraries through
the simple `gem install` command. RubyGems is a part of the Ruby 
programming language. If you have Ruby (version 1.9.2 or later), you have RubyGems.

### Bundler: Tracking Your Project's RubyGems

A companion tool to RubyGems that has become indispensable for Ruby
development is [Bundler](http://gembundler.com/). Bundler is a way
to manage all of the various **gem dependencies** for a Ruby project
simply and in a single place. Bundler is itself a RubyGem and can be
installed at a command line with `gem install bundler`.

With Bundler you can specify all of the dependencies of your application
(the libraries that your application needs to function) in a single
manifest file called a `Gemfile`. Bundler will automatically find
compatible versions of the libraries you specify, and you can additionally
include libraries that are checked out via `git` or are local to your
filesystem.

Bundler is not *mandatory* for creating RubyGems, but it provides
extremely helpful tools for library authors to automate certain 
processes. You will learn more about using Bundler when we create our
first gem.

### So What is A Gem, Anyway?

A RubyGem is essentially a specially structured folder that contains the
source code, description, documentation, and tests of a Ruby library.
Gems can sometimes contain command-line executables as well. The file
structure of a RubyGem usually looks like this:

<pre>
  my-gem
  ├── bin        
  │   └── my-gem 
  ├── lib        
  │   ├── my-gem.rb 
  │   └── my-gem    
  │       └── version.rb
  ├── spec (or test)
  │   ├── spec_helper.rb 
  │   └── my-gem_spec.rb
  ├── my-gem.gemspec
  ├── Gemfile   
  └── Rakefile

</pre>

Let's go through the different parts individually to make sure we know
what they all are:

* `bin`: This directory will contain any command-line executables that
  a gem has defined. For instance, the Bundler gem contains a `bundler`
  executable in this directory.
* `lib`: This directory houses all of your library code. The convention is
  that you should only define a single file at the "root" level of your
  lib directory. That file should be named exactly the same as your gem.
  All other files should go into a subdirectory in the lib directory
  named after your gem (the `lib/my-gem` directory in the example above).
* `spec` or `test`: This directory houses all of the automated tests for
  your gem. Nearly all open source gems have automated tests as testing
  ensures both maximum quality and maximum reliability for code that is
  being worked on by many parties such as an open source project.
* `my-gem.gemspec`: Every gem needs a `gemspec` file that is effectively
  all of the metadata that RubyGems needs to know about your library. We
  will dig into the gemspec in a later post.
* `Gemfile`: This is the manifest file used by Bundler to keep track of
  the project's dependencies. This is primarily present to make it easy
  for other developers to check out and work on the source code of a
  library.
* `Rakefile`: Rake is a library that allows developers to define simple
  "tasks" that can be performed via the `rake` command at the command
  line. Most gems have rake tasks for running specs, generating documentation,
  and performing release-time chores.

And really, that's pretty much it! If you're feeling a little bit
overwhelmed by this list, don't worry: we will cover the needs and uses
of each of these files in depth as we build our gem throughout this
series. I just wanted to give you a clear overview up front of what
really goes into a gem. Remember: a gem is just a folder that follows
some conventions and rules, nothing more.

### Why Would I Want To Create A Gem?

So now you know what a gem is but you may be wondering: why would I
want to create an open source gem? There are many, many reasons why 
individuals and companies alike build open source projects, but here
are some common benefits of open source development:

1. By encapsulating reusable code in a packaged library, you are able
   to reuse that code in multiple applications without rewriting the
   same thing over and over.
2. Individuals who release open source libraries build a reputation in
   the community. Most of the "well-known" Ruby developers became so
   by giving back to the community in the form of useful open source
   libraries.
3. Companies who promote open source development are more likely to
   attract the very best programmers, as the very best programmers in
   the Ruby communities tend to be deeply involved in the open source
   community.
4. When you release open source libraries, other people do some of
   the work for you! When the community can fix bugs and make 
   improvements to common libraries it helps everyone get more done
   faster.

Again, there are far too many reasons to list here as to why you might
want to release open source libraries. You might find some more answers
in Intridea's own [Open Source Citizenry](http://intridea.com/blog/tag/open%20source%20citizenry) blog post series.

In my next post I'll be covering the creation of the basic structure of
a gem as well as getting it well-situated for automated testing, 
documentation, and other common needs. If you have any questions, comments,
inaccuracies, or suggestions please do comment below. I want this to be
a clear and accurate guide that can serve as a great starting point for
anyone who wants to get involved in the Ruby open source world. Until
next time, keep polishing those gems!