+++
title = "Installing Powerline on Mac OS X"
date = "2013-08-19T12:25:00+02:00"
tags = ["MacOS X", "Powerline"]
+++

While installing [Powerline](https://github.com/Lokaltog/powerline) on MacOS X following the [documentation](https://powerline.readthedocs.org/en/latest/installation/osx.html#installation-osx), there are some import problems.

It turns out that running `python setup.py install` installs everything into `/path/to/site-packages/Powerline-beta-py2.7.egg/`. This folder should be named `powerline` instead.

Symlinking the included `powerline` folder to `site-packages/powerline` seems to fix this.
