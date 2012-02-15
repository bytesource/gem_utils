# Gem Utils

### A Rakefile to automate common task during gem development.

The first and only feature so far is finding all dependencies of your gem:

## Finding Gem Dependencies

Gem dependencies are specified in the *gemspec* file with `Gem::Specification.add_dependency`
and `Gem::Specification.add_development_dependency`, respectively [Documentation](http://docs.rubygems.org/read/chapter/20#dependencies).
This Rakefile automates the task of figuring out all dependencies, a task that
that otherwise could be quite time consuming depending on the size of the gem.

## Usage

Move the Rakefile in the same directory as your gem.

The rake task gem:deps can take one or two parameters.

### Examples

The following examples use the gem `rstore` as an example:

``` bash
# List runtime dependencies.
# - First argument: name of the gem

$ rake gem:deps['rstore']
# => ["open-uri", "nokogiri", "bigdecimal", "sequel", "csv"]
```

``` bash
# List development dependencies
# (Gems that are used for development purposes only)
# - First argument: name of the gem
# - Second argument: name of the test folder

$ rake gem:deps['rstore','spec']
# => ["rspec"]

# In this case *sequel* actually requires two additional gems,
# *mysql* and *sqlite3-ruby*, but these dependencies are not revealed,
# before running the test.
```

**Note**: There must *not* be any whitespace between the start of task name and the closing `]`.
          Otherwise Rake will throw an exception.

