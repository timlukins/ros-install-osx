ros-install-osx   
===============

_Updated version of this repo for my attempts at getting ROS running on MacOS Sierra 10.12.5 with XCode 8.8.3 and above._

* Cloned from this excellent effort: https://github.com/tgu/ros-install-osx
* In turn, based on: https://github.com/jundazhu/ros-install-osx
* And the original: https://github.com/mikepurvis/ros-install-osx

**From the original:** This repo aims to maintain a usable, scripted, up-to-date installation procedure for
[ROS](http://ros.org). The intent is that the `install` script may be executed on a
bare Yosemite or El Capitan (_edit:_ and now Sierra) machine and produce a working desktop_full installation,
including RQT, rviz, and Gazebo.

This is the successor to my [popular gist on the same topic][1].

[1]: https://gist.github.com/mikepurvis/9837958

But First...
------------

**Before you start - you should really, really (for you own sanity) remove your current Homebrew installation. This is guaranteed the simplest way to do this! You will then avoid entering a world of pain. A world of pain...**

If you are worried about what you have installed in the past being lost, simply record the output of a `brew list` or `brew cask list` before you start. Then reinstall those formulas at the end (just don't add qt4 or opencv2 whatever you do...)  

How do you properly get rid of Homebrew? As described here: https://github.com/Homebrew/install - the command is:

     /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"

You should also do what it will report to you at the end about removing additional folders (e.g. `/usr/local/*`).

**Also, if you have tried a variant of this or other scripts in the past, you may wish to remove the `/opt/ros` folder that may have been created. You'll probably need `sudo` to do that.** 

*NOTE:* the script will itself re-install Homebrew (and make sure it is at the head of your path) as well as ensuring the crucial Homebrew install python is added as the first thing it does...

Usage
-----

```shell
git clone https://github.com/timlukins/ros-install-osx.git
cd ros-install-osx
./install
```

Troubleshooting
---------------

Here are a few notes on issues overcome (and which may resurface)...

Many of these are similar (but not exactly so) to this post. It proved useful for ideas on how to solve:
https://gist.github.com/plusk01/bb92f6159c0818000865784abfd2a584

As it notes - if you get through the rosdep errors - you simply can work with the remaining catkin commands incrementally (just remember to then to do thremaining commands in the script at the end! All performed in the folder `kinetic_desktop_full_ws`).

### `rosdep` failed with certificate errors 

Because it used the urllib2 library to try and pull package lists over https and it seemed that the classic `<urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed...`. On MacOS Sierra it has it's own certificates in `/etc/ssl` so I ran: `/usr/local/opt/openssl/bin/c_rehash /etc/ssl/`. 

### Rogue CUDA driver installed (even on Mac)

This caused problematic linking with some of the dependencies that then detected CUDA being present. It was simpler just to complete remove it.

### dyld: Library not loaded

If you see this after installation, when trying to execute `rosrun`, then you have [System Integrity Protection](https://support.apple.com/en-us/HT204899) enabled.  The installation script should have detected that and *suggested* a quick fix. 

### QT5...

Manifested when running rosdep as:

	ERROR: the following packages/stacks could not have their rosdep keys resolved to system dependencies:
	webkit_dependency: Cannot locate rosdep definition for [python-qt5-bindings-webkit]

Even if you fix this you will come up against: https://github.com/ros-infrastructure/rosdep/issues/490

So, as suggested by this (and the other error mentioned in the main link above) by skipping the qt5 keys associated with our earlier manual installation of QT5 .

	rosdep install --from-paths src --ignore-src --rosdistro kinetic -y --as-root pip:no --skip-keys="python-qt5-bindings-webkit python-qt-bindings-qwt5 libqt5-core libqt5-gui libqt5-opengl libqt5-opengl-dev libqt5-widgets qt5-qmake qtbase5-dev python-qt-bindings python-qt-bindings-gl python-qt5-bindings python-qt5-bindings-gl"

NOTE: the mechanism for directing rosdep is based on the earlier command that created and updated the contents of the folder:

	/etc/ros/rosdep/sources.list.d/

In here are lists of the packages to install - based on their keys (which is what we use when we say skipkeys)…

e.g. `20-default.list` points to https://raw.githubusercontent.com/jundazhu/ros-install-osx/master/osx-homebrew.yaml which in turn lists entries such as `python-qt5-bindings` etc.
