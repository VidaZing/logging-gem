require 'rainbow/refinement'
using Rainbow
require 'rake/testtask'

GEM_NAME = 'vidazing_logger'

desc "Remove #{GEM_NAME} && Gemfile.lock"
task :clean do
  puts "Remove #{GEM_NAME} && Gemfile.lock".blue
  system "rm -f *.gem Gemfile.lock"
end

desc "Build #{GEM_NAME}"
task :build => :clean do
  puts "Building #{GEM_NAME}".blue
  system "gem build #{GEM_NAME}.gemspec"
end

desc "Installs #{GEM_NAME}"
task :install => :build do
  puts "Installing #{GEM_NAME}".blue
  system "gem install #{GEM_NAME}"
end

desc "Uninstalls #{GEM_NAME}"
task :uninstall do
  puts "UNINSTALLING #{GEM_NAME}".inverse.blue
  system "gem uninstall -xq #{GEM_NAME}"
end

Rake::TestTask.new do |t|
  t.libs << 'test'
  t.test_files = FileList['test/test*.rb']
  t.verbose = true
end

namespace :loop do
  def looper?
    puts "Checking for 'fswatch' to monitor files".blue

    has_fswatch = ! `which fswatch`.empty?

    abort('fswatch is NOT installed. Visit https://github.com/emcrisostomo/fswatch'.bright.red) unless has_fswatch
    puts('fswatch is installed.'.bright.green) if has_fswatch
  end

  def looping(cmd)
    fswatch_cmd = "fswatch -0 -e .git/ -e *.gem -e logs -l 1 ."
    xargs_cmd = "xargs -0 -I {} sh -c \"echo 'File: {}' && %s\""
    looping_cmd = "#{fswatch_cmd} |  #{xargs_cmd}"

    system "#{looping_cmd}" % cmd
  end

  desc "Repeatedly installs the gem on file changes"
  task :install do
    looper?
    puts "Rebuilding on file changes. Ignores .git/, logs/,  and gems created. Watches every 1 seconds".blue
    looping("rake uninstall install")
  end

  desc "Repeatedly runs tests on file changes"
  task :test do
    looper?
    puts "Running tests on file changes. Ignores .git/, logs/, and gems created. Watches every 1 seconds".blue
    looping("rake test")
  end
end
