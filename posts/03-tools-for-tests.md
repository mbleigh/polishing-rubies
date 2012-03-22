# Tools for Tests

A good toolchain is important for any development project. It makes the lives of developers easier by abstracting away or automating repetitive tasks. You should always spend some time at the outset of a project making sure you're using all of the best tools available. On an open source project this is 10 times more important.

Instead of building a toolchain that will be used by your company or small team, you're building a toolchain that will potentially be used by dozens, hundreds, or even *thousands* of other developers. An annoyance that costs five minutes, multiplied by all those developers, equals hours and hours of lost potential productivity. Now that we've bootstrapped the directory structure of our gem, it's time to get our toolchain up and running so that development for you (and all of your contributors some day) is as painless and fast as possible.

### Test Setup

Having tests for your gem is fundamentally important. If you have no tests, you have no way to vet incoming pull requests, check for regressions between versions, or really have any control over the quality of your project. Luckily, the Ruby community has a strong testing culture so there are great tools to use and widespread adoption of testing practices.

I'm going to walk through how to set your gem up for testing via [RSpec](http://github.com/rspec/rspec). In general the Ruby community nearly universally uses either `RSpec` or the built-in `Test::Unit` as its framework of choice. I would love to cover `Test::Unit` as well, but I don't have enough of a grasp on best practices to be able to tell you how to do that here. If you would like to contribute such a section to this guide, by all means please [fork it on GitHub](https://github.com/mbleigh/polishing-rubies) and I'll be happy to update the content.

The first step to using RSpec in your tests is to add it to your project's gemspec:

```ruby
gem.add_development_dependency 'rspec', '~> 2.9'
```
This tells RubyGems that RSpec is a necessary library for developers who are going to try to contribute to your gem. Once you've added the dependency, run `bundle` to update your dependencies with Bundler. Note that because Bundler is capable of reading dependencies from a `gemspec` you don't need to manually specify them in your `Gemfile`.

With RSpec, it's a good practice to specify the most recent **minor version** (the second number in the `MAJOR.MINOR.PATCH` versioning system) for your dependency. This way if RSpec updates to a new major version that breaks existing functionality, you can review any breakages and fix them with a new release of your gem rather than having unstable tests in the wild.

Now that you have RSpec in your project's bundle, you should run `bundle exec rspec --init`. This is a simple generator that will bootstrap your project's RSpec setup. The output should look something like this:

```txt
create   spec/spec_helper.rb
create   .rspec
```

The `.rspec` file is an options file that lets you specify the default command-line options (such as output format and color) that will be used by RSpec's runner in your project. If you've written tests before, you should be familiar with `spec/spec_helper.rb` as a central place to bootstrap your test suite and add any dependent library requires, etc.

Because we're writing a gem we will want to set up our `spec_helper.rb` file to work with our gem's structure. Alter the file to look like this:

```ruby
$:.unshift File.dirname(__FILE__) + '/../lib'
require 'my-gem'

RSpec.configure do |config|
  config.treat_symbols_as_metadata_keys_with_true_values = true
  config.run_all_when_everything_filtered = true
  config.filter_run :focus
end
```

The first line adds our gem's `lib` directory to Ruby's **load path**. This means that when we say something like `require 'some-file'` Ruby will automatically look in our gem's `lib` directory to see if that file exists. The second line, `require 'my-gem'`, is the same line that your end users will type when they want to start using your gem. This allows us to write tests from the perspective of someone who has installed the gem and required it, either manually or automatically through a tool like Bundler.

Now that we have our `spec_helper` all set up, let's go ahead and create our first spec. Create a file `spec/my_gem_spec.rb` (replace `my_gem` with your gem's name) and put this in it:

```ruby
require 'spec_helper'

describe MyGem do
  it 'requires additional testing'
end
```

Now simply run the `rspec` command in your project's root directory. You should see output that looks like this:

```txt
Run options: include {:focus=>true}

All examples were filtered out; ignoring {:focus=>true}
*

Pending:
  MyGem requires additional testing
    # Not yet implemented
    # ./spec/my_gem_spec.rb:4

Finished in 0.00015 seconds
1 example, 0 failures, 1 pending
```

Congratulations! You've got your gem's test suite up and running.

### Testing With Rake

Another standard practice amongst all Ruby projects (apps and gems alike) is to make it possible to run the tests for a project via a Rake task. This is remarkably easy; all you need to do is open up the `Rakefile` in your project's directory and add the following to the bottom of your `Rakefile`:

```ruby
require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new
task :default => :spec
```

This does two things: first, it creates a new RSpec rake task that, by default, can be run by typing `rake spec` at your project's root. Second, we assign the `:default` task to be `:spec`. This means that in your project's root you can simply run `rake` and all of the tests for your project will run. This is a convention in the Ruby community and a good practice for all projects.

To verify everything we've done we can simply run `rake -T` which is a command to list the available Rake tasks. You should see something like this:

```txt
rake build    # Build my-gem-0.0.1.gem into the pkg directory
rake install  # Build and install my-gem-0.0.1.gem into system gems
rake release  # Create tag v0.0.1 and build and push my-gem-0.0.1.gem to Ru...
rake spec     # Run RSpec code examples
```

As you can see, we have four tasks available. Three of them are provided to us by Bundler for packaging and releasing our gem, and one of them is provided by RSpec for testing our gem.

### Autotesting With Guard

[Guard](https://github.com/guard/guard) is a gem that allows you to observe your project's files for change and perform actions based on that change. The first thing you will need to do is add Guard (and its helpers for Bundler and RSpec) to the end of your `Gemfile`:

```ruby
gem 'guard'
gem 'guard-rspec'
gem 'guard-bundler'
```

Note that, differently from RSpec, I prefer to add Guard and its related gems to my project's `Gemfile`, not its `gemspec`. This choice comes down to personal preference, but while RSpec is **required** for someone to do development on my gem, Guard is really more of a **nice to have** and so I relax the dependency somewhat. The effect is the same for contributors: simply running `bundle` in the project's directory will install all of the dependencies from the `gemspec` *and* the `Gemfile`. Speaking of which, you should now run `bundle` in your project root to install Guard if it isn't already.

Now we need to initialize Guard and configure it for our project. Luckily, Guard comes with its own command line helpers:

```txt
guard init
guard init bundler
guard init rspec
```

Guard works similarly to Bundler and Rake by creating a `Guardfile` in your project's root directory. These commands automatically add example configuration for each guard type to the Guardfile (after `guard init` creates the file to begin with). While I'm going to tell you explicitly what to put in your `Guardfile`, the `init` commands can really help jog your memory if you're trying to do it from scratch. Let's modify our `Guardfile` to look like this:

```ruby
guard 'bundler' do
  watch('Gemfile')
  watch(/^.+\.gemspec/)
end

guard 'rspec', :version => 2 do
  watch(%r{^spec/.+_spec\.rb$})
  watch(%r{^lib/(.+)\.rb$})     { |m| "spec/#{m[1]}_spec.rb" }
  watch('spec/spec_helper.rb')  { "spec" }
end
```

Guard works by watching certain files in your project and then performing actions when those files change. The first guard tells us to watch for the `Gemfile` and the `gemspec` to change, and re-bundle the application when that happens. The second guard tells us to watch all of our RSpec test files, our `spec_helper`, and the *corresponding library files* for each of our RSpec tests. This means that when you change a file, Guard can automatically re-run the tests for that specific file only.

Now that your `Guardfile` is properly configured, you can just run `bundle exec guard` from your project's root directory. This will get Guard up and running and you will now automatically run tests and re-bundle as you build your gem. Running Guard makes the development feedback loop as tight and automatic as possible.

### Continuous Integration

Continuous Integration is the process of automatically running the tests for a project when new code is pushed to the central repository. Luckily, a team of developers has built [Travis CI](http://travis-ci.org) that provides **free** continuous integration for open source projects of all types.

Why do you need continuous integration for an open source project?

* Travis allows you to test your code **across multiple versions of Ruby**. Even if you don't personally develop on older versions of Ruby or alternative implementations such as JRuby or Rubinius, Travis can run your tests on these Rubies and notify you of any build failures.
* Travis gives you the confidence to merge pull requests from the web. If someone submits a pull request you can merge it into your repository knowing that if it breaks your test build Travis will let you know. In the future, Travis will even support testing pull requests *before* you merge them.

In addition to the benefits provided by Travis is it exceptionally easy to implement. Here's how:

1. Go to [http://travis-ci.org](http://travis-ci.org) and sign in with your GitHub account. Travis needs access to your GitHub info to be able to set up the automatic build process for each repository.
2. Hover over your name once logged in and go to your profile. This lists out your open source repositories; from here you can simply "switch on" CI for each project.
3. Set up a `.travis.yml` file in your project.


We haven't yet created a GitHub repository for our project, so steps 1 and 2 will actually come a bit later. However, we can set up the configuration file so that when we're ready to push to GitHub we will get continuous integration immediately. To set up Travis for your project you need to create one more configuration file in your repository. Make a file called `.travis.yml` (the leading dot is not a typo) and add this to it:

```yaml
rvm:
  - 1.9.3
  - 1.9.2
  - jruby
  - rbx
script: "bundle exec rake"
```

As you can see, the configuration is very straightforward. The `rvm` key allows us to specify the different *versions of Ruby* we want to test against. The `script` key tells Travis what command to run for the build. Any command that exits with a non-zero status code (a Unix standard for a command that had errors) will automatically fail the build in Travis. In our case we're going to run `bundle exec rake` which, as explained above, has been aliased to running our RSpec tests via Rake.

Once you've added your Travis configuration file, it's time to create another commit:

```txt
git add .
git commit -m "Adds RSpec, Rake testing, Guard, and Travis."
```

All right! Now we've not only created a gem but we've added robust tools for testing that will give us and other contributors the power to develop quickly and in a test-driven manner. We've also set up continuous integration to let us test against many different versions of Ruby, even if we don't have them installed! We're well on our way to building a gem. Stay tuned for the next installment!