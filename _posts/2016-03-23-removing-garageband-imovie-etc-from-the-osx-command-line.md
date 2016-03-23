---
layout: post
title: Removing GarageBand, iMovie, etc. from the OSX command line
categories: osx garageband imovie
---

I was getting annoyed with an offer to update GarageBand, iMovie, Keynote, Numbers, and Pages since I never use these apps. While it is possible to install another app to remove these I would rather just remove them from the command line.

## Manually

With some guidance from <http://www.tekrevue.com/tip/delete-garageband/> here are the commands I used to remove Garage Band:

{% highlight bash %}
sudo rm -rf /Applications/GarageBand.app
sudo rm -rf /Library/Application\ Support/GarageBand
sudo rm -rf /Library/Application\ Support/Logic/
sudo rm -rf /Library/Audio/Apple\ Loops
sudo rm -rf /Library/Audio/Apple\ Loops\ Index
{% endhighlight %}

I removed the others from `/Applications` using the following commands:

{% highlight bash %}
sudo rm -rf /Applications/iMovie.app
sudo rm -rf /Applications/Keynote.app
sudo rm -rf /Applications/Numbers.app
sudo rm -rf /Applications/Pages.app
{% endhighlight %}

According to <https://howtouninstallonmac.net/how-to-uninstall-imovie-on-mac-os-x/> we should check some other locations such as `ls ~/Library/Preferences` and `ls ~/Library/Application Support` but I could not see anything relevant to remove from these locations.

I saved over 9GB which is significant for systems with smaller SSD drives.

## Reinstall

According to <https://support.apple.com/kb/PH18868> it should be possible to reinstall these apps from the Mac App Store.

## Future idea

I would like to see an open-source Homebrew utility script that can remove each of these apps automatically. A bonus would be a GUI wrapper that can show these commands running. Credit for these ideas goes to Ubuntu.
