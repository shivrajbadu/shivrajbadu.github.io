---
layout: post
title:  "How to install Ruby on Windows?"
date:   2018-08-6 04:23:58 +0545
categories: [Ruby, Operate on WindowsOS]
tags: [ruby, windowsOS]
---

### When you are on Windows machine

##### You can install BitnamiRubyStack Installers or RubyInstaller. But BitnamiRubyStack always doesnot have latest ruby supported for Win.

##### When you are dealing with RubyEncoder you might need the same Ruby Version in which application is build. So here is the tips how you can switch your ruby versions in your Win Machine.
###### Let say by BitnamiRubyStack your ruby version is already installed to 2.0.x ver and it is deafault version used in your system. Also you had already installed Ruby 2.4.x version using RubyInsaller but it is not the default just installed. So what you need to do is:

###### - Uninstall default ruby 2.0
###### - load the installed ruby 2.4.x bin executable path
###### - check from any dir say c:/>ruby -v , it should display ruby 2.4.2
###### - Now everything is Ok to move ahead

### You may get following errors:

#### ERR:

```
C:\Users\Siv\Desktop\2314\newtest>bundle install
C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/dependency.rb:308:in `to_specs': Could not find 'bundler' (>= 0) among 13 total gem(s) (Gem::MissingSpecError)
Checked in 'GEM_PATH=C:/Users/Siv/.gem/ruby/2.4.0;C:/Ruby24-x64/lib/ruby/gems/2.4.0', execute `gem env` for more information
        from C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/dependency.rb:320:in `to_spec'
        from C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/core_ext/kernel_gem.rb:65:in `gem'
```

#### To Resolve just run

```ruby
gem install bundler
```

#### ERR:

```
C:\Users\Siv\Desktop\2314\newtest>rails s
C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/dependency.rb:308:in `to_specs': Could not find 'railties' (>= 0) among 14 total gem(s) (Gem::MissingSpecError)
Checked in 'GEM_PATH=C:/Users/Siv/.gem/ruby/2.4.0;C:/Ruby24-x64/lib/ruby/gems/2.4.0', execute `gem env` for more information
        from C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/dependency.rb:320:in `to_spec'
        from C:/Ruby24-x64/lib/ruby/2.4.0/rubygems/core_ext/kernel_gem.rb:65:in `gem'
        from C:/Ruby200/bin/rails:22:in `<main>'
```
#### Just run following commands

```ruby
bundle install
```

#### Add the gem

Please add the following to your Gemfile to avoid polling for changes:

```ruby
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
```

{% include inarticle-adsense.html %}