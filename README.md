# cncjs-shopfloor-tablet

## fork
# small changes for nice display on official RsPi LCD 800x480
 !!! pre ALPHA STAGE !!!

### A stable cncjs UI for machine operators

The normal cncjs UI is too complex for production use on a shop floor.  The many widgets, small buttons, complex screen repositioning, and developer-level information displays get in the way of a machine operator's needs for predictability and ease of access to the most important and frequently-used functions.

The cncjs-pendant-* UIs are great for simple jogging, but lack functions needed for running a complete job.

This project builds on cncjs-pendant-tinyweb to create a UI suitable for running production jobs.  It adds the following capabilities to cncjs-pendant-tinyweb:

* Loading GCode files from the cncjs server's watch directory
* Text display of the currently loaded GCode file
* Two MDI (Manual Data Input) boxes to run arbitray GCode commands
* Goto specific coordinates
* Set the work coordinates to specified values (not just zero)
* Direct buttons (bypassing the dropdown) for setting the jog increment
* Automatic reconnect to most-recently-used serial port
* State-driven highlighting of applicable program-control buttons
* Separation between important buttons to reduce accidental touches
* Removal of semi-dangerous buttons like diagonal jogs (diagonal jogging requires the operator to monitor two simultaneous motions, which pushes human attention limits)
* 2D toolpath display with tool position tracking

The layout is optimized for use on tablet computers.  It works well on inexpensive tablets, even those with a slow GPU, since it does not use 3D graphics.

It can be used as the only UI for running complete jobs - choosing the program, setting up the work coordinate system, and controlling the run.

The full cncjs UI can still be used, perhaps on a different computer or in a different browser tab on the same computer, if its advanced capabilities are required for something that cncjs-shopfloor-tablet does not support.  The intention is that cncjs-shopfloor-tablet is sufficient for daily use in a production environment.

### Limitations

It works well with TinyG/g2core and GRBL.  It has been tested a little with Marlin but not extensively.  It has not been tested with Smoothieware - but Smoothie and GRBL are quite similar from a protocol standpoint so it is likely to work.

Rotary axes A/B/C are not supported from the GUI, but you can issue GCode commands for those axes manually commands via the MDI boxes.

### Setup

Get the cncjs-shopfloor-tablet files onto the machine control computer that runs the cncjs app, either by cloning the git tree or by downloading and extracting a .zip.

Create a subdirectory to contain GCode files, for example "/home/pi/GCode".

```
$ mkdir /home/pi/GCode
```

Tell cnc to use that directory as the "watch directory" by adding a line like this to the .cncrc file, after the opening brace:

```
    "watchDirectory": "/home/pi/GCode",
```

Use cnc's -m option to set up a static mount.  Assuming that the files are in the directory */home/pi/cncjs-shopfloor-tablet*, the command would be:

```
$ cncjs -m /tablet:/home/pi/cncjs-shopfloor-tablet/src
```

If that command fails with an EADDRINUSE message, the problem is that cnc is already running and you will need to kill the old process before restarting with the shopfloor option.

If you are using pm2 to autostart cncjs, use this command sequence:

```
$ pm2 stop $(which cncjs)
$ pm2 delete $(which cncjs)
$ pm2 start $(which cncjs) -- -m /tablet:/home/pi/cncjs-shopfloor-tablet/src
$ pm2 save
```

If you are using cron to autostart cncjs, you can kill the old cncjs process with ```$ killall node```, then manually run the ```$cncjs ...``` command above for testing.  To make cncjs-shopfloor-tablet work after rebooting, use ```$ crontab -e``` to edit the start instructions, adding ```-m /tablet:/home/pi/cncjs-shopfloor-tablet/src``` to the *@reboot* line.  The line might look like:
```
@reboot /usr/bin/cncjs -m /tablet:/home/pi/cncjs-shopfloor-tablet >> $HOME/cncjs.log 2>&1 &
```

After you have restarted the server with the -m option, browse to the url 'http://*host*:8000/tablet/', where *host* is the name or IP address of the cncjs server.

You can still use the full cncjs UI by browsing 'http://*host*:8000'. You can use the different UI's simultaneously in different browser windows or tabs, which can be on different machines if you wish.

### Usage

![cncjs-tablet 2](https://user-images.githubusercontent.com/4861133/33970662-4a8244b2-e018-11e7-92ab-5a379e3de461.PNG)

* **Start/Pause/Resume/Stop** are highlighted and colored according to the program state
    * **Start** is green when it is possible to start running a program
    * **Pause** is red when the program is running
    * **Stop** is red when the program is running or paused
    * **Resume** is green when the program is paused
* The **Inch** or **mm** button shows the currently-active units, and toggles them if clicked.
* **X=** **Y=** **Z=** set the axis work coordinate to the value in the number box above
* **GoX** **GoY** **GoZ** rapid to the axis work coordinate to the value in the number box above
* **X=0** **Y=0** **Z=0** set the axis work coordinate to 0
* **GoX0** **GoY0** **GoZ0** rapid to work 0 for that axis
* **0.001** .. **5** set the jog increment.
* The selector box between **Z+** and **Z-** shows the current jog increment, and, when clicked, permits the choice of some additional jog increments.
* **X-** **X+** **Y-** **Y+** **Z-** **Z+** jog by the current increment.  You can jog continuously by selecting a large increment, starting the jog, then hitting **Stop** when it has gone far enough.
* **MDI** sends the GCode command entered in the box to its left.  To resend that command, click **MDI** again.  There are MDI blocks so you can have two different GCode commands "queued up" for easy execution.
* To load a GCode file from the cncjs server's watch directory, select it from the file selector at the lower left.  Its GCode text will be displayed in the scrollable textarea to the right and the X-Y projection of its toolpath will be displayed in the image area to the right of that.
* **Rfrsh** refreshes the file selector list.  That is useful if additional files are added to the watch directory.  Another way to refresh the file list is to reload the page.
* **Load** reloads the GCode program from the currently selected file.  That is useful if you edit the file from another computer and want to pick up the new version.  To reload it directly from the file selector, you would have to first select a different file and then re-select the edited one (because of the way selector ".change" events work).
* If the file selector is blank and there is GCode text in the GCode display text area, the GCode is probably a macro loaded by a different cncjs session.
