namespace :gem do
  desc "List all Ruby files of a gem ('rstore/lib' by default)"
  task :list_files, :gem_name, :sub_folder do |t, args|
    args.with_defaults(:gem_name => '', :sub_folder => 'lib')
    gem    = args[:gem_name]
    folder = args[:sub_folder]
    raise Exception, "Name of gem not specified" if gem.empty?

    path = File.join('.', gem, folder)
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
      raise Exception, "Gem '#{gem}' not found. \nMake sure the gem folder and the Rakefile are in the same directory."
    end
  end

  desc "List all dependencies"
  task :deps, [:gem_name, :sub_folder] => ["gem:list_files"] do |t, args|
    # Remove references to files required with a path, such as
    # require 'to/path/name'
    # These are references to internal files, not to external gems.
    file_sep = Regexp.new(Regexp.escape("/"))
    files_required = @with_require.reject {|line| line =~ file_sep }
    files_required

    # Extract gem name
    files_required.map! do |gem|
      gem.match(/('|")(?<name>.+)('|")/)[:name]
    end

    # Remove duplicates
    files_required.uniq!

    # Finally remove all gems with the same name as one of the source files listed above.
    # They are not external gems, but normal source files located in /lib or /spec (e.g. 'spec_helper')
    # who are on the load path ($:). Files in these folder are 'required' without a path and
    # therefore were not removed using #reject above.
    files_on_path = @ruby_files.reject {|path| path =~ file_sep }
    external_gems = files_required.reject {|name| files_on_path.any? {|file| name == File.basename(file, '.rb') } }
    p external_gems
  end
end
