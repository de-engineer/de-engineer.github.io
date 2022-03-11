source "https://rubygems.org"
gem 'jekyll-archives'
gem 'jekyll-toc'
gemspec
gem 'wdm', '~> 0.1.1', :install_if => Gem.win_platform?
# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# # and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
gem "tzinfo", ">= 1", "< 3"
gem "tzinfo-data"
end
gem "webrick" 
