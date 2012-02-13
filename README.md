# Gem Utils

### A Rakefile to automate common task during gem development.

The first and only feature so far is finding all dependencies of your gem:

## Finding Gem Dependencies

Gem dependencies are specified in the *gemspec* file with `Gem::Specification.add_dependency`
and `Gem::Specification.add_development_dependency`, respectively.
This Rakefile automates the task of figuring out all dependencies, a task that
that otherwise could be quite time consuming depending on the size of the gem.

## Usage

Move the Rakefile in the same directory as your gem.

The rake task gem:deps can take one or two parameters.

### Examples

The following examples use a I wrote as an example:

``` bash
# Find dependencies of the gem 'rstore'

$ rake gem:deps['rstore']
# => ["open-uri", "nokogiri", "bigdecimal", "sequel", "csv"]
```

``` bash
# Find development dependencies
# (pass the name of the test folder as the second argument)

$ rake gem:deps['rstore','spec']
# => ["rspec", "pp", "csv", "sequel"]
```

**Note**: There must not be any whitespace between the start of task name and the closing ].
          Otherwise Rake will throw an exception.

