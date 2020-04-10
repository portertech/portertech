---
title: "Homebrew + RVM > Awesome"
date: 2010-03-26T16:19:04-07:00
draft: false
toc: false
images:
tags: 
  - homebrew
  - ruby
  - rvm
  - gem
  - rubygems
  - osx
  - apple
---

I recently opened a box containing my new MacBook Pro. All of my files are synced/backed up with [Dropbox](http://www.dropbox.com/), allowing me to install Dropbox on the new computer and leave it for a couple of hours. The MBP is my development machine, so I needed all of my tools installed with the ability to update them with ease. In the past, I used MacPorts to take care of my MySQL, Memcached, and Ruby installions and it worked just fine. This time around however, I wanted something new and fun. Homebrew.

Homebrew is a new package manager for OS X. Unlike Fink or MacPorts, Homebrew integrates with the core operating system, reducing the number of extra libraries to install etc. Another neat feature is the ability to write software package recipes in Ruby, awesome.

Here are some raw installation instructions (clean system). I like to keep everything under user ownership to make life more enjoyable, say no to sudo.

You will need the latest version of xcode, you can get it [here](http://developer.apple.com/technologies/xcode.html). After the installation is complete, you may continue.

```
sudo mkdir /usr/local
sudo chown -R $USER /usr/local
curl -Lsf \
http://github.com/mxcl/homebrew/tarball/master | \
tar xvz -C /usr/local --strip 1
```
Homebrew is installed and "/usr/local" is already part of your path, now to give it a go.

```
brew install git
brew update
brew install wget
brew install mysql
brew install memcached
brew install postgres
brew install redis
brew install mongodb
brew list
```

My homebrew experience has been a happy one, I don't see myself using MacPorts or Fink again.

Homebrew documentation can be found [here](http://wiki.github.com/mxcl/homebrew/installation).

RVM (Ruby Version Manager) takes care of managing my multiple Ruby and Gem versions. Leaving the OS X default installation of ruby alone, RVM installs to a users home directory, changing the required system variables. Just how Homebrew is under user ownership, RVM is quite happy running this way as well and so am I.

To install RVM, do the following.

```
mkdir -p ~/.rvm/src && cd ~/.rvm/src && rm -rf ./rvm && \
git clone --depth 1 git://github.com/wayneeseguin/rvm.git && \
cd rvm && ./install
```

Add the following line to the end of your .profile or .bash_profile or .bashrc file.

```
if [[ -s $HOME/.rvm/scripts/rvm ]]; then
  source $HOME/.rvm/scripts/rvm;
fi
```

Try out RVM.

```
rvm install 1.8.7
rvm install 1.9.2
rvm --default 1.9.2
ruby -v
which ruby
gem install rails
```

Pretty neat eh? (I'm Canadian, it's ok)

RVM documentation can be found [here](http://rvm.beginrescueend.com/).
