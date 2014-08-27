require "fileutils"
require "rubygems"

require "mail_catcher/version"

require "bundler/gem_tasks"

module Bundler
  class GemHelper
    unless method_defined?(:rubygem_push)
      raise NoMethodError, "Monkey patching Bundler::GemHelper#rubygem_push failed: did the Bundler API change???"
    end

    def rubygem_push(path)
      sh %{gem inabox #{path}}

      Bundler.ui.confirm "Pushed #{name} #{version}"
    end
  end
end

# XXX: Would prefer to use Rake::SprocketsTask but can't populate
# non-digest assets, and we don't want sprockets at runtime so
# can't use manifest directly. Perhaps index.html should be
# precompiled with digest assets paths?

desc "Compile assets"
task "assets" do
  compiled_path = File.expand_path("../public/assets", __FILE__)
  FileUtils.mkdir_p(compiled_path)

  require "mail_catcher/web/assets"
  sprockets = MailCatcher::Web::Assets
  sprockets.css_compressor = :sass
  sprockets.js_compressor = :uglifier
  sprockets.each_logical_path(/(\Amailcatcher\.(js|css)|\.(xsl|png)\Z)/) do |logical_path|
    if asset = sprockets.find_asset(logical_path)
      target = File.join(compiled_path, logical_path)
      asset.write_to target
    end
  end
end

desc "Package as Gem"
task "package" => ["assets"] do
  require "rubygems/package"
  require "rubygems/specification"

  spec_file = File.expand_path("../mailcatcher.gemspec", __FILE__)
  spec = Gem::Specification.load(spec_file)

  Gem::Package.build spec
end

require "rdoc/task"

RDoc::Task.new(:rdoc => "doc",:clobber_rdoc => "doc:clean", :rerdoc => "doc:force") do |rdoc|
  rdoc.title = "MailCatcher #{MailCatcher::VERSION}"
  rdoc.rdoc_dir = "doc"
  rdoc.main = "README.md"
  rdoc.rdoc_files.include "lib/**/*.rb"
end

require "rake/testtask"

Rake::TestTask.new do |task|
  task.pattern = "spec/*_spec.rb"
end

task :test => :assets

task :default => :test
