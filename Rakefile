#
# Requirements Block...
#
require 'rubygems'
require 'bundler/setup'
require 'rake'
require 'yaml'
require 'jekyll'
require 'launchy'
require 'html-proofer'

#
# Define Variable...
#
$options = {
    "source"      => File.expand_path("."),
    "destination" => File.expand_path("_site"),
    "watch"       => true,
    "serving"     => true
}

#
# Create Methods...
#
def build()
    puts "Compiling the site..."
    Jekyll::Commands::Build.process($options)
end

def serve()
    puts "Running Jekyll..."
    Jekyll::Commands::Serve.process($options)
end

def testsite()
    puts "Testing Website..."
    HTMLProofer.check_directory("./_site", {
        :allow_hash_href  => true,
        :assume_extension => true,
        :check_favicon    => true,
        :check_html       => true,
        :check_img_http   => true,
        :enforce_https    => true,
        :href_ignore      => [
            "http://127.0.0.1",
            "http://127.0.0.1/",
            "http://127.0.0.1/icingaweb2/",
            "http://127.0.0.1/icingaweb2/setup"
        ],
        :only_4xx         => true,
        :verbose          => true,
        :typhoeus         => {
            :verbose      => false
        },
        :parallel         => {
            :in_processes => 3
        }
    }).run
end

#
# Default Task Specification
#
task :default => [:serve]

task :build do
    build
end

task :preview do
    build
    Thread.new do
        sleep 2
        puts "Opening in browser..."
        Launchy.open("http://127.0.0.1:4000")
    end
    serve
end

task :serve do
    build
    serve
end

task :test do
    build
    testsite
end
