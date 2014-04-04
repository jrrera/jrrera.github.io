---
layout: post
title: "Installing Jekyll on OS X 10.9.2"
description: ""
category: ""
tags: ["Jekyll", "OS X 10.9.2", "RVM"]
---

{% include JB/setup %}

Ironically, as I was trying to install Jekyll to set up this blog on Github, I ran into a lot of problems. The command-line error wasn't too helpful but it looked something like this:

    Building native extensions.  This could take a while...
    ERROR:  Error installing jekyll:
    	ERROR: Failed to build gem native extension.
    
        /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby extconf.rb
    creating Makefile
    
    make "DESTDIR=" clean
    
    make "DESTDIR="
    compiling porter.c
    porter.c:359:27: warning: '&&' within '||' [-Wlogical-op-parentheses]
          if (a > 1 || a == 1 && !cvc(z, z->k - 1)) z->k--;
                    ~~ ~~~~~~~^~~~~~~~~~~~~~~~~~~~
    porter.c:359:27: note: place parentheses around the '&&' expression to silence this warning
          if (a > 1 || a == 1 && !cvc(z, z->k - 1)) z->k--;
                              ^
                       (                          )
    1 warning generated.
    compiling porter_wrap.c
    linking shared-object stemmer.bundle
    clang: error: unknown argument: '-multiply_definedsuppress' [-Wunused-command-line-argument-hard-error-in-future]
    clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
    make: *** [stemmer.bundle] Error 1
    
    make failed, exit code 2
    
    Gem files will remain installed in /Library/Ruby/Gems/2.0.0/gems/fast-stemmer-1.0.2 for inspection.
    Results logged to /Library/Ruby/Gems/2.0.0/extensions/universal-darwin-13/2.0.0/fast-stemmer-1.0.2/gem_make.out

Both the [Jekyll troubleshooting page](http://jekyllrb.com/docs/troubleshooting/) and StackOverflow recommended two ways to resolve installation issues:

 - Making sure [commandline tools](https://developer.apple.com/downloads/index.action?name=for%20Xcode%20-#) was installed
 - Making sure RubyGems was up to date with `sudo gem update --system`

However, neither of these did the trick for me. Thankfully, after an hour of searching, I ended up finding the right clues [here](http://stackoverflow.com/questions/22479246/how-to-install-jekyll-on-mac-osx-10-9-with-ruby-2-0-0).

As per the StackOverflow link above, I installed Homebrew and RVM, [using this guide](http://dean.io/setting-up-a-ruby-on-rails-development-environment-on-mavericks/); this did the trick. Success!

While I do wonder why RVM / Homebrew was necessary, it seems to be working for others as well. Thought I'd call it out for any others struggling out there, as I had difficulty finding the right clues on Stack Overflow.