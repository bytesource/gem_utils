namespace :internal do
  desc "List all Ruby files of a gem ('rstore/lib' by default)"
  task :list_files, :gem_name, :sub_folder do |t, args|
    args.with_defaults(:gem_name => '', :sub_folder => 'lib')
    @gem    = args[:gem_name]
    @folder = args[:sub_folder]
    raise Exception, "Name of gem not specified" if @gem.empty?

    path = File.join('.', @gem, @folder)
    # Destination gem folder must at the top level of the current directory
    if Dir.exists?(path)
      Dir.chdir(path) do
        # Recursively gather the paths to each Ruby file
        @ruby_files = Dir['**/*.rb']

        @with_require = []
        @ruby_files.each do |path|
          File.open(path) do |f|
            f.each_line do |line|
              # We are only interested in lines containing the key word 'require'.
              next  unless line =~ /^\s*require/
              @with_require << line
            end
          end
        end
      end
    else
      raise Exception, "Gem '#{@gem}' not found. \nMake sure the gem folder and the Rakefile are in the same directory."
    end
  end

  desc "List all dependencies"
  task :deps, [:gem_name, :sub_folder] => ["list_files"] do |t, args|
    # Remove references to files required with a path, such as
    # require 'to/path/name'
    # These are references to internal files, not to external gems.
    # file_sep = Regexp.new(Regexp.escape("/"))
    file_sep = /\//
    files_required = @with_require.reject {|line| line =~ file_sep }

    # Extract gem name
    files_required.map! do |gem|
      gem.match(/('|")(?<name>.+)('|")/)[:name]
    end

    # Remove duplicates
    files_required.uniq!

    # Finally remove all gems with the same name as any of the source files listed above.
    # These are not external gems, but normal source files located in /lib or /spec (e.g. 'spec_helper')
    # who are on the load path ($:). Files in these folder are 'required' without a path and
    # therefore were not removed using #reject above.
    files_on_path = @ruby_files.reject {|path| path =~ file_sep }
    # Remove filename suffix to allow for a direct comparison in the last step
    files_on_path.map! {|name_and_suffix| File.basename(name_and_suffix, '.rb') }
    @external_dependencies = files_required.reject {|name| files_on_path.include?(name) }

    # Remove all gems that are part of Ruby's standard library
    # Fetch gem names:
    require 'open-uri'
    require 'nokogiri'
    url = 'http://www.ruby-doc.org/stdlib-1.9.3/toc.html'

    libs = Nokogiri::HTML(open(url)).css('a.mature').map do |node|
      node.text
    end

    @external_dependencies = @external_dependencies.reject { |gem| libs.include?(gem) }

    @external_dependencies
  end
end

namespace :gem do
  desc "List dependencies (handle development dependencies special)"
  task :deps, [:gem_name, :sub_folder] => ["internal:deps"] do |t, args|

    if @folder == 'lib'
      p @external_dependencies
    else
      all_development_dependencies = @external_dependencies
      # Development dependencies by definition are gems that are used for development purposes ONLY.
      # Hence we have to remove those gems, that are already included in the default gems under
      # the /lib folder.
      # To do this we run the above tasks again, this time searching /lib. Then we can exclude
      # any gems from 'all_development_dependencies' that are also part of the default dependencies.

      # Rake::Task.reenable resets the task's already_invoked state,
      # allowing the task to then be executed again.
      # Note: All tasks that are to be run again (either directly by calling 'invoke' or
      # indirectly as part of the dependency chain) need to be re-enabled,
      # because dependencies already invoked are not re-executed.
      Rake::Task["internal:list_files"].reenable
      Rake::Task["internal:deps"].reenable
      Rake::Task["internal:deps"].invoke(@gem, 'lib')
      # After invoking the above task with the default folder name 'lib' again,
      # @external_dependencies now equals the default dependencies.
      runtime_dependencies = @external_dependencies
      development_dependencies = all_development_dependencies.reject {|gem| runtime_dependencies.include?(gem) }
      p development_dependencies
    end
  end
end

# Conclusion:
# The code for re-running a task looks kind of messy.
# Next time I might consider writing these tasks as normal methods.
