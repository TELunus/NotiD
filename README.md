# NotiD
NoitD is a daemon that prints notifications from the [Desktop Notifications](http://www.galago-project.org/specs/notification/0.9/index.html) speficiation to it's STDOUT.

NotiD is forked from [statnot](http://github.com/halhen/statnot/) by [Henrik Lindberg](https://github.com/halhen)
## Background
An old piece of UNIX philosophy is that each program should do exactly one thing well, then you can connect multiple programs together to create advanced functionality in a versitile way.  However most notification daemons want to do two things:

1. listen for desktop notifications
2. show a popup window with the contents of that notification

NotiD is designed to only listens for desktop notifications and prints them to STDOUT, letting something else handle making a popup (or doing something entirely different with the notification!).

### Desktop Notifications
If you have used a "regular" window manager like KDE or Gnome, you have probably come across notifications. The are typically small windows with text messages and sometimes an icon that shows for a couple of seconds before they fade out. They are for example used to let the user know that a new instant message has arrived or that the battery is running low. Desktop Notifications is a specification created for freedesktop.org that many applications use. For example Pidgin and Evolution can be configured to notify for new messages using Desktop Notifications.

## Installation

To install NotiD, first install the required dependencies:

* [python 2.5+](http://www.python.org)
* [dbus-python](http://dbus.freedesktop.org/releases/dbus-python/)
* [pygtk](http://www.pygtk.org/) - (not for GUI support, but the dbus-python library requires it)

Next, adjust the target directories in the `config.mk` file to fit your setup. 

To install, run as root:

    # make install

Finally, NotiD needs to start with the window manager. You can for example add the following to .xinitrc:

    killall notification-daemon &> /dev/null
    NotiD & 

Note that the statnot needs to be the only notification tool running. The example above makes sure that `notification-daemon` is not running.

## Configuration

For advanced configuration, a configuration file can be passed to statnot on the command line, which overrides the default settings. This configuration file must be written in valid python, which will be read if the filename is given on the command line.  You do only need to set the variables you want to change, and can leave the rest out.

Below is an example of a configuration which sets the defaults.

    # Default time a notification is show, unless specified in notification
    DEFAULT_NOTIFY_TIMEOUT = 3000 # milliseconds
    
    # Maximum time a notification is allowed to show
    MAX_NOTIFY_TIMEOUT = 5000 # milliseconds
    
    # Maximum number of characters in a notification. 
    NOTIFICATION_MAX_LENGTH = 100 # number of characters
    
    # Time between regular status updates
    STATUS_UPDATE_INTERVAL = 2.0 # seconds
    
    # Command to fetch status text from. We read from stdout.
    # Each argument must be an element in the array
    # os must be imported to use os.getenv
    import os
    STATUS_COMMAND = ['/bin/sh', '%s/.statusline.sh' % os.getenv('HOME')] 
     
    # Always show text from STATUS_COMMAND? If false, only show notifications
    USE_STATUSTEXT=True
     
    # Put incoming notifications in a queue, so each one is shown.
    # If false, the most recent notification is shown directly.
    QUEUE_NOTIFICATIONS=True

    # update_text(text) is called when the status text should be updated
    # If there is a pending notification to be formatted, it is appended as
    # the final argument to the STATUS_COMMAND, e.g. as $1 in default shellscript
     
    # dwm statusbar update
    import subprocess
    def update_text(text):
        # Get first line
        first_line = text.splitline()[:-1]
        subprocess.call(["xsetroot", "-name", first_line])

## Possible errors
If notifications are not shown, make sure that no other notification-daemon is running. `killall notification-daemon` is a good command to try. Restart NotiD if there was another daemon running. Also make sure NotiD is running

## Supported software
More and more applications use Desktop Notifications. Use [Google](http://www.google.com) to find solutions for your applications. `libnotify` is a good term to search, since it is a common library used by many applications.

You can also send your own notifications to NotiD. This is easily done with the `notify-send` command. For example, `notify-send "Hello World"` will print `Hello World` in the status bar according to your speficiation. This is useful to notify that a long running task like a download or software build has finished.

notify-send can also be used for other, more direct messages. For exampe, you could call a script called `dwm-volume` when your volume media buttons on the keyboard are pressed. This script adjusts the volume and sends a notification containing e.g. `vol [52%] [on]`. 

    #!/bin/sh
    if [ $# -eq 1 ]; then
        amixer -q set Master $1
    fi
    notify-send "`amixer get Master | awk 'NR==5 {print "vol " $4, $6}'`"

## Final notes
NotiD is currently released under the GPL 2.0 because that was the liscense statnot used.  Really I prefer BSD or MIT liscening, so perhaps at some point I'll rewrite NotiD from scratch and give it a new liscence. Bug reports or feature requests for NotiD can be left as issues on github (feature requests may go unimplemented as the whole point is a simple lightweight tool that does one thing well).


Notid was created by TELunus
The original authors of statnot (from which NotiD is forked) were:
 * Henrik Hallberg  (<halhen@k2h.se>); halhen@github
 * Olivier Ramonat; enzbang@github
