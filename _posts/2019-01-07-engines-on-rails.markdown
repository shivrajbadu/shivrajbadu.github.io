---
layout: post
title:  "Engine on Rails"
date:   2019-01-07 08:01:18 +0545
categories: tutorials
---

Engines are small applications which provides functionality to their host applications. A Rails application is a engine with `Rails::Application` class inheriting a lot of behaviour from `Rails::Engine`. So Rails application and engine are alomost same thing and share common structure with slight differences.

Engines also related with plugins, both shares `lib/` directory structure, and both generated using `rails plugin new` generator. Engine is considered as full plugin using `--full` in generator while `--mountable` option includes features of full and some others. An engine can be a plugin and plugin can be an engine.

Some example of engines are: `Devise` for authenticating parent application, `Thredded` for forum functionlity, `Spree` for ecommerce, `Refinery CMS` for CMS application.

Create a Base application (e.g: base_app)

Create engine e.g: engine_app with following command:
```
  rails new plugin engine_app --mountable
```

Customize engine_app.gemspec file and edit, homepage, summary, description, etc as per your requirements.

Go to base_app -  Gemfile
```
  gem 'engine_app', path: 'lib/engine_app'
  bundle install
```

When everything is Ok, we will get some message like:
Using engine_app 0.0.1 from source at lib/engine_app

How to make plugins -  routes easy?

Go to routes.rb file of base app and paste the code:
```
mount EngineApp::Engine, at: '/engine_app'
```

or

Go to plugins app/lib/engine_app/engine.rb

```
isolate_namespace EngineApp

initializer "engine_app", before: :load_config_initializers do |app|
      Rails.application.routes.append do
          mount EngineApp::Engine, at: '/engine_app'
      end
end
```

Now it is possible to run rake routes and we can see all engines routes

```
e.g: engine_app /engine_app EngineApp::Engine
```

How to check Plugins Rails Console?

```
Go to main app
> rails console
> EngineApp::Article.new
```

How to make MVC in base and inside engines?

To make model controller inside base go to base dir or else go to engine dir

##### Rails Engine Migrations

Base app has no knowledge of Engine migrations, we need to customize it or manually we need to run command:

i) Manually we can run following command:
```
rake engine_app:install:migrations
```

ii) go to lib/engine.rb file and paste the following code
 ```
 initializer "engine_app", before: :load_config_initializers do |app|
     config.paths["db/migrate"].expanded.each do |expanded_path|
         Rails.application.config.paths["db/migrate"] << expanded_path
     end
 end
 ```

Now `rake db:migrate` will work from base application

How to access plugins and base app's controller action?

For e.g: `link_to 'Home', root_path`, will not work if we want to access engines
resources because engine won't able to understand root_path for it. So we need following codes:

```
link_to 'Home', main_app.root_path
link_to 'Plugin Home', engine_app.articles_path
```

How to layout in base and engines?

We have to call layouts inside application controller or wherever required and it can be accessed by:

```
layouts 'application' # will call base layouts
layouts 'engine_app/application' # will call plugin layouts
```

How to include gems inside engines?

Go to .gemspec file e.g: engine_app.gemspec

```
Gem::Specification.new do |s|
  s.add_dependency "devise"
  s.add_dependency 'authority', '~> 3.1'
end
```
