require 'thor/group'
require 'erb'

module Middleman
  class Generator < ::Thor::Group
    include ::Thor::Actions

GENERAL_CONFIG = <<-eos
# Reload the browser automatically whenever files change
activate :directory_indexes
activate :asset_hash
eos

HELPERS_CONFIG = <<-eos
# Methods defined in the helpers block are available in templates
helpers do
  def nav_link(text, path)
    opts = {}
    opts[:class] = 'active' if current_page.path == path
    link_to text, path, opts
  end
end
eos

BUILD_CONFIG = <<-eos
configure :build do
  # Minify CSS on build
  activate :minify_css

  # Minify Javascript on build
  activate :minify_javascript
end
eos

DISTRIBUTION_CONFIG = <<-eos
activate :s3_sync do |s3_sync|
  s3_sync.bucket                     = '<%= @AWS_BUCKET %>' # The name of the S3 bucket you are targeting. This is globally unique.
  s3_sync.region                     = '<%= @AWS_REGION %>'     # The AWS region for your bucket.
  s3_sync.aws_access_key_id          = '<%= @AWS_ACCESS_ID %>'
  s3_sync.aws_secret_access_key      = '<%= @AWS_ACCESS_KEY %>'
  s3_sync.delete                     = true # We delete stray files by default.
  s3_sync.after_build                = true # We do not chain after the build step by default.
  s3_sync.prefer_gzip                = true
  s3_sync.path_style                 = true
  s3_sync.reduced_redundancy_storage = false
  s3_sync.acl                        = 'public-read'
  s3_sync.encryption                 = false
  s3_sync.prefix                     = ''
  s3_sync.version_bucket             = false
  s3_sync.index_document             = 'index.html'
  s3_sync.error_document             = '404.html'
end

activate :cloudfront do |cf|
  cf.access_key_id = '<%= @AWS_ACCESS_ID %>'
  cf.secret_access_key = '<%= @AWS_ACCESS_KEY %>'
  cf.distribution_id = '<%= @CLOUDFRONT_DISTRIBUTION_ID %>'
  cf.filter = /\.html$/i  # default is /.*/
  cf.after_build = true  # default is false
end
eos

GEMS_CONFIG = <<-eos
gem 'middleman', '>= 4.0.0'
gem 'middleman-livereload'
gem 'middleman-cloudfront', git: 'https://github.com/andrusha/middleman-cloudfront'
gem 'middleman-s3_sync'
gem 'mime-types'
eos

    source_root File.expand_path(File.dirname(__FILE__))

    def copy_default_files
      directory 'template', '.', exclude_pattern: /\.DS_Store$/
    end

    def ask_for_aws_credentials
      @AWS_BUCKET = ask 'AWS_BUCKET'
      @AWS_REGION = ask 'AWS_REGION'
      @AWS_ACCESS_ID = ask 'AWS_ACCESS_ID', echo: false
      @AWS_ACCESS_KEY = ask 'AWS_ACCESS_KEY', echo: false
      @CLOUDFRONT_DISTRIBUTION_ID = ask 'CLOUDFRONT_DISTRIBUTION_ID', echo: false
    end

    def build_gemfile
      insert_into_file 'config.rb', GENERAL_CONFIG, after: "# General configuration\n"
      insert_into_file 'config.rb', HELPERS_CONFIG, after: "# Helpers\n"
      insert_into_file 'config.rb', BUILD_CONFIG, after: "# Build-specific configuration\n"
      insert_into_file 'config.rb', ERB.new(DISTRIBUTION_CONFIG).result(binding), after: "# Distribution configuration\n"

      insert_into_file 'Gemfile', GEMS_CONFIG, after: "# Middleman Gems\n"
    end
  end
end

