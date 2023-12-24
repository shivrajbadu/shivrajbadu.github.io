---
layout: post
title:  "Pow alternatives prax !"
date:   2019-01-05 09:01:18 +0545
categories: [Ruby, Pow Alternatives Prax]
tags: [ruby, linux, prax, pow_alternate, rack gem]
---

Pow is zero-config Rack server for Mac OS X. Your application will run on myapp.test without modifying /etc/hosts.
Those who use GNU/Linux and installed Ruby and Rack gem Prax is usefull. It is a web server which start rack application in background and proxy all requests to that application.

#### Configurations

```
  git clone git://github.com/ysbaddaden/prax.git /opt/prax
  cd /opt/prax/ && ./bin/prax install
  sudo /etc/init.d/prax start
  # Go to application and run the command
  # cd apps/yourappname
  prax link
  # open your application with this command
  prax open
  # or
  google-chrome http://yourappname.dev/

  # see the list of linking application using
  prax list
```

If you are using (RVM) Ruby version manager, follow below steps:

```
cd $HOME
touch .praxconfig
```
Paste this code in .praxconfig file
```
# detect `$rvm_path`
if [ -z "${rvm_path:-}" ] && [ -x "${HOME:-}/.rvm/bin/rvm" ]
then rvm_path="${HOME:-}/.rvm"
fi
if [ -z "${rvm_path:-}" ] && [ -x "/usr/local/rvm/bin/rvm" ]
then rvm_path="/usr/local/rvm"
fi

# load environment of current project ruby
if
  [ -n "${rvm_path:-}" ] &&
  [ -x "${rvm_path:-}/bin/rvm" ] &&
  rvm_project_environment=`"${rvm_path:-}/bin/rvm" . do rvm env --path
2>/dev/null` &&
  [ -n "${rvm_project_environment:-}" ] &&
  [ -s "${rvm_project_environment:-}" ]
then
  echo "RVM loading: ${rvm_project_environment:-}"
  \. "${rvm_project_environment:-}"
else
  echo "RVM project not found at: $PWD"
fi
```

When your host example.dev do not work, then you need to restart your application using prax:

```
# Go to home directory and cd into .prax
cd .prax
# Go to your application directory
cd example.dev
# for first time run the command `prax start`, later you can restart it
prax restart
```
