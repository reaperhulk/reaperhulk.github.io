---
author: justfalter
comments: true
date: 2011-06-16 18:49:14+00:00
layout: post
slug: how-to-install-qcachegrind-kcachegrind-on-mac-osx-snow-leopard
title: How to install qcachegrind (kcachegrind) on Mac OSX Snow Leopard
wordpress\_id: 1501
---

_Guest post by Mike! I had originally posted this as a [gist](https://gist.github.com/1029580) on github, but Paul [selflessly insisted](https://twitter.com/#!/reaperhulk/status/81395560162136064) that I blog about it, so here you go.___

I like to use [kcachegrind](http://kcachegrind.sourceforge.net/html/Home.html) for doing profiling of my ruby code via [ruby-prof](http://github.com/rdp/ruby-prof). This works fantastically if you're on a platform that has KDE installed. However, most of my development is done on OSX, and while you can install kcachegrind via macports, it takes hours and hours because it has to build KDE, as well. Much to my surprise, the fine folks who wrote kcachegrind also made a QT version, qcachegrind. I was able to build this on OSX in about 10 minutes without too much effort, only having to install QT and GraphViz. Yippie!

I should note that I'm running OSX 10.6.7, with Xcode 4. My default gcc/g++ version is 4.2. I'm sure it will build just fine on earlier versions of Xcode, but I haven't tested it.


### Step 1: Install QT


You could do this via "brew install qt" if you have [homebrew](https://github.com/mxcl/homebrew), but this takes a long time, so I downloaded the binary distribution of version 4.7.3 for OSX 10.5/10.6 from [this page](http://qt.nokia.com/downloads/qt-for-open-source-cpp-development-on-mac-os-x/). Installing is easy: open up the dmg, and install QT via the mpkg file contained within.


### Step 2: Install Graphviz


Second, you should install [Graphviz](http://www.graphviz.org/) so that qcachegrind can generate pretty call graphs for you. Like before, you could install this via "brew install graphviz", but this also takes a really long time and I'm super impatient, so I just downloaded the latest binary installer from [this page](http://www.graphviz.org/Download_macos.php).


The only component of Graphviz that qcachegrind uses, to the best of my knowledge, is the _dot_ binary. By default, the Graphviz installer drops _dot_ to _/usr/local/bin/dot_, which qcachgrind.app might not find since /usr/local/bin isn't in PATH by default (at least, not on my system). In order to ensure that qcachegrind can find _dot_, I created a symlink from /usr/bin/dot to /usr/local/bin/dot:




```
sudo ln -s /usr/local/bin/dot /usr/bin/dot
```

###Step 3: Building qcachegrind

This is actually simple. Check out the source code for kcachegrind and switch to the qcachegrind directory:

```
svn co svn://anonsvn.kde.org/home/kde/trunk/KDE/kdesdk/kcachegrind kcachegrind
cd kcachegrind/qcachegrind
```

You build the application by running 'qmake', a QT utility that can generate a boring-old Makefile, if we ask it nicely enough. By default, qmake wanted to build an xcode project on my box. I pointed it at spec profile that generates a Makefile that will build using g++, and things seemed to work fine. You can see all the profiles available in this directory: /usr/local/Qt4.7/mkspecs. This command also managed to spit out a whole bunch of errors and warnings, yet everything seemed to build properly.

```
qmake -spec 'macx-g++'
make
```

###Step 4: Enjoy!
You should now have a beautiful little "qcachegrind.app" sitting in your build directory. You can run this directly via "open qcachegrind.app", or you can drop it in your /Applications directory for easy access.

###Update (Aug 30, 2011):
* I removed the step where I would apply a patch that added the ability to open any file name. The qcachegrind source has since been updated to include a functionally identical change.

* There are several comments about installing Qt via brew, and trying to build qcachegrind using that, yet failing. John Rood, looking to enable other gluttons for punishment, suggested installing Qt like so will get you on your way ```brew install qt --with-qt3support``` (I haven't tested this out, myself).

* For those who have installed Qt via brew and are having problems: try uninstalling it if you're not using it for anything else ('brew uninstall qt'). You may or may not have to reinstall the binary version, afterward; I'm not sure.
