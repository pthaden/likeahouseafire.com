---
layout: post

title: "When `yum groupinstall` Says Group Does Not Exist"

excerpt: "A workaround when your Linux repos don't have groups configured and you can't just create them"

tags: [JCS]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

We've got some Linux VMs that point at a private set of repositories for Yum updates.  Recently I went to build an system and wanted to add a graphical desktop.  When I executed `yum groupinstall Desktop` this error came back:

{% highlight bash %}

Warning: Group Desktop does not exist.
Error: No packages in any requested group available to install or update
{% endhighlight %}

Turns out it's [not uncommon to have private repos that haven't had their groups created](http://unix.stackexchange.com/questions/118394/local-yum-repository-with-grouplist-not-working). Sure enough, a `yum grouplist` showed no groups available:

{% highlight bash %}

Error: No group data available for configured repositories
{% endhighlight %}

The problem here is: I don't own the repos and can't update their configuration with groups, and our policy is that I couldn't just add another external repo for the VM to use.  But, something I *did* have access to was another Linux VM whose repos pointed to the standard Unbreakable Linux Network.

On that machine, I could list out all the components of the `Desktop` group with the `yum groupinfo` command:

{% highlight bash %}

yum groupinfo Desktop

...

Group: Desktop
 Description: A minimal desktop that can also be used as a thin client.
 Mandatory Packages:
   NetworkManager
   NetworkManager-gnome
   alsa-plugins-pulseaudio
   at-spi
   control-center
   dbus
   gdm
   gdm-user-switch-applet
   gnome-panel
   gnome-power-manager
   gnome-screensaver
   gnome-session
   gnome-terminal
   gvfs-archive
   gvfs-fuse
   gvfs-smb
   metacity
   nautilus
   notification-daemon
   polkit-gnome
   xdg-user-dirs-gtk
   yelp
 Default Packages:
   control-center-extra
   eog
   gdm-plugin-fingerprint
   gnome-applets
   gnome-media
   gnome-packagekit
   gnome-vfs2-smb
   gok
   openssh-askpass
   orca
   pulseaudio-module-gconf
   pulseaudio-module-x11
   rhn-setup-gnome
   vino
 Optional Packages:
   sabayon-apply
   tigervnc-server
   xguest
{% endhighlight %}

Usually a `yum groupinstall` grabs [just the Mandatory and Default Packages](http://sapiengames.com/2014/05/18/install-optional-packages-yum-groupinstall-command/) and installs them all in turn. Copying and pasting this list of packages into a editor made it easy to multi-select the CR/LF and tabs and replace them with a single space.  Then it was easy to create a "non-group" install command:

{% highlight bash %}
yum install -y NetworkManager NetworkManager-gnome alsa-plugins-pulseaudio at-spi control-center dbus gdm gdm-user-switch-applet gnome-panel gnome-power-manager gnome-screensaver gnome-session gnome-terminal gvfs-archive gvfs-fuse gvfs-smb metacity nautilus notification-daemon polkit-gnome xdg-user-dirs-gtk yelp control-center-extra eog gdm-plugin-fingerprint gnome-applets gnome-media gnome-packagekit gnome-vfs2-smb gok openssh-askpass orca pulseaudio-module-gconf pulseaudio-module-x11 rhn-setup-gnome vino

{% endhighlight %}

This worked great because all those packages were in my private repo; it was only the group definitions that were missing. Once I ran that big command (and a few more for the other groups I wanted to install), my VM was running a GUI login just fine.