require 'rake'
require 'rake/testtask'
require 'rake/rdoctask'

WYSIHAT_ROOT     = File.expand_path(File.dirname(__FILE__))
WYSIHAT_SRC_DIR  = File.join(WYSIHAT_ROOT, 'src')
WYSIHAT_DIST_DIR = File.join(WYSIHAT_ROOT, 'dist')
WYSIHAT_DOC_DIR  = File.join(WYSIHAT_ROOT, 'doc')

desc "Update git submodules"
task :update_submodules do
  system("git submodule update --init")
end

task :default => :dist

desc "Builds the distribution."
task :dist => :update_submodules do
  require File.join(WYSIHAT_ROOT, "vendor", "sprockets", "lib", "sprockets")

  Dir.chdir(WYSIHAT_SRC_DIR)

  environment  = Sprockets::Environment.new(".")
  preprocessor = Sprockets::Preprocessor.new(environment)

  %w(wysihat.js).each do |filename|
    pathname = environment.find(filename)
    preprocessor.require(pathname.source_file)
  end

  output = preprocessor.output_file
  File.open(File.join(WYSIHAT_DIST_DIR, "wysihat.js"), 'w') { |f| f.write(output) }
end

desc "Empties the output directory and builds the documentation."
task :doc => ["doc:clean", "doc:build"]

namespace :doc do
  desc "Builds the documentation"
  task :build => :update_submodules do
    require File.join(WYSIHAT_ROOT, "vendor", "pdoc", "lib", "pdoc")
    files = Dir["#{File.expand_path(File.dirname(__FILE__))}/src/**/*.js"]
    files << { :output => WYSIHAT_DOC_DIR }
    PDoc::Runner.new(*files).run
  end

  desc "Empties documentation directory"
  task :clean do
    rm_rf WYSIHAT_DOC_DIR
  end
end

desc "Builds the distribution, runs the JavaScript unit tests and collects their results."
task :test => [:build_tests, :dist, :test_units]

require 'vendor/jstest'
desc "Runs all the JavaScript unit tests and collects the results"
JavaScriptTestTask.new(:test_units) do |t|
  testcases        = ENV['TESTCASES']
  tests_to_run     = ENV['TESTS']    && ENV['TESTS'].split(',')
  browsers_to_test = ENV['BROWSERS'] && ENV['BROWSERS'].split(',')

  t.mount("/dist")
  t.mount("/test")

  Dir["test/unit/*.html"].sort.each do |test_file|
    tests = testcases ? { :url => "/#{test_file}", :testcases => testcases } : "/#{test_file}"
    test_filename = test_file[/.*\/(.+?)\.html/, 1]
    t.run(tests) unless tests_to_run && !tests_to_run.include?(test_filename)
  end

  %w( safari firefox ie konqueror opera ).each do |browser|
    t.browser(browser.to_sym) unless browsers_to_test && !browsers_to_test.include?(browser)
  end
end

task :build_tests => [:clean_test_files] do
  Dir["test/unit/*_test.js"].each do |test_file|
    TestBuilder.new(test_file).render
  end
end

task :clean_test_files do
  Dir["test/unit/*_test.html"].each do |test_file|
    FileUtils.rm_rf(test_file)
  end
end
