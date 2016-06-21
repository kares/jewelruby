source 'https://rubygems.org'

if ENV['GH_PAGES'] == 'true'
  require 'json'
  require 'open-uri'
  versions = JSON.parse(open('https://pages.github.com/versions.json').read)

  gem 'github-pages', versions['github-pages']
else
  gem 'github-pages'
end

gem 'jekyll-seo-tag'
gem 'jekyll-sitemap'

gem 'therubyracer', require: nil

# NOTE: < 1.10 https://github.com/jekyll/jekyll/issues/3970
# and 1.6.2 as https://github.com/jekyll/jekyll/pull/2951
# gem 'rouge', '<= 1.6.2', require: nil # syntax highlighter
