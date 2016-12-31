+++
title = "Building ZSH with zpython module on MacOS X"
date = "2013-08-19T12:19:00+02:00"
tags = ["MacOS X", "ZSH", "Python"]
+++

There is a fork of ZSH that has a Python module. It is able to run [Powerline](https://github.com/Lokaltog/powerline) smoothly.

However, on MacOS X, it doesn't correctly detect the Python libs upon compilation. There seems to be a similar problem for Vim with python support: http://stackoverflow.com/questions/6490513/vim-failing-to-compile-with-python-on-os-x

The workaround is the same. Editing the `Makefile` in the `config` directory of the Python installation fixes this. The modification is `PYTHONFRAMEWORKINSTALLDIR` instead of `PYTHONFRAMEWORKDIR` on the line beginning with `LINKFORSHARED`
