# frozen_string_literal: true

require 'rainbow/refinement'
using Rainbow
require 'rake/testtask'
require 'coveralls/rake/task'

$LOAD_PATH.push File.expand_path('lib', __dir__)
require 'vidazing_logger/version'
VERSION = VidazingLogger::VERSION

GEM_NAME = 'vidazing_logger'

# IMPORTANT: Color can't be used for `system` commands.
GEM_NAME_VERSION = "#{GEM_NAME}-#{VERSION}"
GEM_ARTIFACT = "#{GEM_NAME_VERSION}.gem"

desc "Remove #{GEM_ARTIFACT} && Gemfile.lock"
task :clean do
  puts "Removing #{GEM_ARTIFACT} && Gemfile.lock".blue
  system 'rm -f *.gem Gemfile.lock'
end

desc "Build #{GEM_NAME_VERSION}"
task build: :clean do
  puts "Building #{GEM_NAME_VERSION}".blue
  system "gem build #{GEM_NAME}.gemspec"
end

task default: ['build']

desc "Publish #{GEM_ARTIFACT}"
task :publish do
  puts "Publishing #{GEM_NAME_VERSION}".blue
  system "gem push #{GEM_NAME_VERSION}.gem"
end

desc "Installs #{GEM_ARTIFACT}"
task install: :build do
  puts "Installing #{GEM_ARTIFACT}".blue
  system "gem install #{GEM_NAME}"
end

desc "Uninstalls #{GEM_ARTIFACT}"
task :uninstall do
  puts "UNINSTALLING #{GEM_ARTIFACT}".inverse.blue
  system "gem uninstall -xq #{GEM_NAME}"
end

namespace :doc do
  desc "Build documentation into 'doc/'"
  task :build do
    puts "Building documentation into 'doc/':".blue
    system 'yard'
  end

  desc 'Find undocumented code'
  task :coverage do
    puts 'Documentation coverage:'.blue
    system 'yard stats --list-undoc'
  end

  desc 'See local documentation at http://localhost:8808'
  task :serve do
    system 'yard server --reload'
  end
end

Rake::TestTask.new do |t|
  t.libs << 'test'
  t.test_files = FileList['test/test*.rb']
  t.verbose = true
end

namespace :loop do
  def looper?
    puts "Checking for 'fswatch' to monitor files".blue

    has_fswatch = !`which fswatch`.empty?

    abort('fswatch is NOT installed. Visit https://github.com/emcrisostomo/fswatch'.bright.red) unless has_fswatch
    puts('fswatch is installed.'.bright.green) if has_fswatch
  end

  def looping(cmd)
    fswatch_cmd = 'fswatch -0 -e .git/ -e *.gem -e logs -e .yardoc -l 1 .'
    xargs_cmd = "xargs -0 -I {} sh -c \"echo 'File: {}' && %s\""
    looping_cmd = "#{fswatch_cmd} |  #{xargs_cmd}"

    system format(looping_cmd.to_s, cmd)
  end

  IGNORED_MESSAGE = 'Ignores .git/, logs/, .yardoc/, and gems created. Watches every 1 seconds'

  desc 'Repeatedly installs the gem on file changes'
  task :install do
    looper?
    puts "Rebuilding on file changes. #{IGNORED_MESSAGE}".blue
    looping('rake uninstall install')
  end

  desc 'Repeatedly runs tests on file changes'
  task :test do
    looper?
    puts "Running tests on file changes. #{IGNORED_MESSAGE}".blue
    looping('rake test')
  end

  desc 'Repeatedly show documentation coverage on file changes'
  task :"doc:coverage" do
    looper?
    puts "Showing undocumented code on file changes. #{IGNORED_MESSAGE}".blue
    looping('rake doc:coverage')
  end
end

Coveralls::RakeTask.new
