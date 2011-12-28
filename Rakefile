# encoding: utf-8

require 'yaml'

##
# Load Nanoc3 tasks for additional validators
#
# require 'nanoc3/tasks'

##
# Configurable Constants
#
BASE_URL = 'http://developer-dailymile-com.herokuapp.com/'

##
# Heroku-based Deployment
#
# Requires a Heroku Application (a heroku remote) that runs on Celadon Cedar
#
desc 'Deploy the website to Heroku using Git.'
task :deploy do
  prepare!
  compile!
  Rake::Task["optimize:all"].invoke
  deploy!
  revert!
end

##
# A couple of rake tasks that'll optimize JPG, PNG, JavaScripts and Stylesheet files
#
namespace :optimize do

  ##
  # Gem Requirement:
  #  YUI-Compressor - Bundled in Gemfile
  #
  desc 'Compress all stylesheet files'
  task :stylesheets do
    require 'yui/compressor'
    compressor = YUI::CssCompressor.new

    Dir['output/**/*.css'].each do |stylesheet|
      puts "Compressing Stylesheet: #{stylesheet}"
      css = File.read(stylesheet)
      File.open(stylesheet, 'w') do |file|
        file.write(compressor.compress(css))
      end
    end
  end

  ##
  # Gem Requirement:
  #  YUI-Compressor - Bundled in Gemfile
  #
  desc 'Compress all javascript files'
  task :javascripts do
    require 'yui/compressor'
    compressor = YUI::JavaScriptCompressor.new(:munge => true)

    Dir['output/**/*.js'].each do |javascript|
      puts "Compressing JavaScript: #{javascript}"
      js = File.read(javascript)
      File.open(javascript, 'w') do |file|
        file.write(compressor.compress(js))
      end
    end
  end

  ##
  # Package Requirement:
  #  jpegoptim
  # Install OSX:
  #  brew install jpegoptim
  # Install Ubuntu:
  #  [apt-get | aptitude] install jpegoptim
  #
  desc 'Optimize JPG images in output/images directory using jpegoptim'
  task :jpg do
    puts `find output/images -name '*.jpg' -exec jpegoptim {} \\;`
  end

  ##
  # Package Requirement:
  #  optipng
  # Install OSX:
  #  brew install optipng
  # Install Ubuntu:
  #  [apt-get | aptitude] install optipng
  #
  desc 'Optimize PNG images in output/images directory using optipng'
  task :png do
    puts `find output/images -name '*.png' -exec optipng {} \\;`
  end

  ##
  # Convenient task for performing all optimization techniques at once
  #
  desc 'Optimize all JPG, PNG, Stylesheet and JavaScript files in the output directory'
  task :all => [:jpg, :png, :javascripts, :stylesheets]
end

##
# Use this method to change the base url in the configuration file
# so you can automate that instead of manually changing it everytime
# when you want to deploy the website
#
def change_base_url_to(url)
  puts "Changing Base URL to #{url}.."
  config = YAML.load_file('./config.yaml')
  config['base_url'] = url
  File.open('./config.yaml', 'w') do |file|
    file.write(config.to_yaml)
  end
end

##
# Re-compile by removing the output directory and re-compiling
#
def compile!
  puts "Compiling website.."
  puts %x[rm -rf output]
  puts %x[nanoc compile]
end

##
# Prepares the deployment environment
#
def prepare!
  %x[git checkout master]
  unless %x[git status] =~ /nothing to commit \(working directory clean\)/
    puts "Please commit your changes on the master branch before deploying!"
    exit 1
  end

  puts "Creating and moving in to \"deployment\" branch.."
  puts %x[git checkout -b deployment]

  puts "Removing \"output\" directory from .gitignore.."
  gitignore = File.read(".gitignore")
  File.open(".gitignore", "w") do |file|
    file.write(gitignore.gsub("output", ""))
  end

  change_base_url_to(BASE_URL)
end

##
# Moves back to the "master" branch and removes the "deployment" branch
#
def revert!
  %x[git checkout master]
  %x[git branch -D deployment]
end

##
# Deploys the application via git to Heroku
#
def deploy!
  puts "Adding and committing compiled output for deployment.."
  puts %x[git add .]
  puts %x[git commit -a -m "temporary commit for deployment"]
  puts 'Deploying to Heroku..'
  puts %x[git push heroku HEAD:master --force]
end
