# Gem Utils

### A Rakefile to automate common task during gem development.

The first and only feature so far is finding all dependencies of your gem:

## Finding Gem Dependencies

Gem dependencies are specified in the *gemspec* file with `Gem::Specification.add_dependency`
and `Gem::Specification.add_development_dependency`, respectively [Documentation](http://docs.rubygems.org/read/chapter/20#dependencies).
This Rakefile automates the task of figuring out all dependencies, a task that
can be quite time consuming and failure prone if done manually.

## Usage

1. `$ gem install 'nokogiri'` (used to fetch the gem names of the Ruby Standard Library)
2. Move the Rakefile in the same directory as your gem.
3. Call the Rake task `gem:deps` with one or two parameters (see examples for details):

### Examples

The following examples use the gem `rstore` as an example:

**Listing all runtime dependencies**

First argument: Name of the gem

``` bash
$ rake gem:deps['rstore']
# => ["nokogiri", "sequel"]
```
_Note_: Gem names that are part of the [Ruby 1.9 Standard Library](http://www.ruby-doc.org/stdlib-1.9.3/) are automatically
        removed from the output.

**Listing all development dependencies**
(Gems used for development purposes only)

First argument:  Name of the gem
Second argument: Name of the test folder

``` bash
$ rake gem:deps['rstore','spec']
# => ["rspec"]

# In this case *sequel* actually requires two additional gems,
# *mysql* and *sqlite3*, but these dependencies are internal to *sequel*
# and not revealed until actually running the tests.
```

**Note**: There must *not* be any whitespace between the start of task name and the closing `]`.
          Otherwise Rake will throw an exception.

