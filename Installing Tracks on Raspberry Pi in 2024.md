Installing Tracks on Raspberry Pi in 2024
========================

TLDR : [Tracks installation for RPi3B+](https://github.com/FixedCamel/tracks_FC/blob/main/Installing%20Tracks%20on%20Raspberry%20Pi%20in%202024.md#3-software-installation).

## 1. Introduction


After many years looking into productivity solutions I found [Tracks](https://www.getontracks.org/) (Web hosted somewhere) about a decade ago, and used it quite a lot for some time.

Then I stayed away for a few years and when I returned that option was no longer available.

So after exploring the different ways to use it again I finally got it to work on a[ Raspberry Pi 3 Model B+](https://www.raspberrypi.com/products/raspberry-pi-3-model-b-plus/) or **RPi3B+** for short.

Reasons:
1. I didn't want to pay for [Taskitin.fi](https://www.taskitin.fi/) at this point and for the next reason.
2. I am not a fan of someone else having access to my data.
3. The [TurnKey Linux](https://www.turnkeylinux.org/tracks) route had all of the above issues and more. I don't know if it even works at the moment.
4. I tried Docker but faced several issues:
   * I am not a fan of Docker in general. Having to install and run 3 layers really bugs me.
   * I have managed to stay away from Docker and have little experience.
   * Thus the issues I faced with the official [Docker image](https://hub.docker.com/r/tracksapp/tracks) were too much.
   * [Staannoe's Docker image](https://github.com/TracksApp/tracks/wiki/Staannoe%27s-Docker-image) is not available to use anymore.
   * I don't mind having access to Tracks only when my computer is on but having it always running and accessible from any device in my network is pretty cool!

I chose to install from scratch using the official installation instructions for a [Custom Server Installation](https://github.com/TracksApp/tracks/blob/master/doc/installation.md#custom-server-installation).

However I had to make some deviations from those instructions that took quite some time. Some because I didn't know Ruby at all :-) 

I am writing this guide to help my future self, help others and contribute to this awesome project.

None of the links are sponsored.

Kudos to everyone that has contributed so far, especially [original developer bsag](http://www.rousette.org.uk/projects), design and UI developer T. Martens and current maintainer ZeiP! Thank you all so much!

Looking forward to your feedback to improve this guide.

I am not a software developer but I do have a particular set of skills, and writing this is quite relaxing. Enjoy :-)

---

## 2. Raspberry Pi
### Hardware

I already had a [ Raspberry Pi 3 Model B+](https://www.raspberrypi.com/products/raspberry-pi-3-model-b-plus/) lying around with a 32GB SD card. I have no doubt that you can substitute for newer version and maybe older ones.

Here some parts that I would recommend if I did it all over again:
* [Cool case and power supply](https://www.amazon.com/dp/B07BTHNW9W)
* [Highly recommended SD card for RPi3](https://www.amazon.com/dp/B06XYHN68L)

### Software

RPi3B+ needs the old installation method, the new one does not require a second computer, but now one can use [Raspberry Pi Imager](https://www.raspberrypi.com/software/) .

Using the **Imager** I chose the newest **32bit Raspberry Pi OS with desktop** edited the settings disabling the Pi Remote desktop just because.

![OS_Selection](./2c.png)

Then connect everything and power up. This whole process is pretty straight forward and only took minutes till the Pi desktop appeared. I removed some programs and updated some Pi software.

A proper power source is important for the Pi's performance. Even an official power supply can fail after many years. Any answer that is not all zeros (0x0) is bad.

![Complete set of choices for installation.](./2d.png)


I highly recommend to check your Pi for throttling using this command in the console.

`vcgencmd get_throttled`

![Checking for thottle](./1_000.png)

---

##  3. Software Installation

In the following subsections I present the instructions for installing Tracks on a fresh 32bit Raspberry Pi OS using SQLite3.
I reviewed this three times because the first two I over-installed some prerequisites or overused sudo. Depending on feedback I will update this guide.

It's separated in seven subsections:
1. Installing Ruby on Rails
2. Installing nodejs & yarn
3. Installing individual gems
4. Editing Tracks Gemfile
5. Tracks installation
6. Starting the server
7. Adding to autostart

### Installing Ruby on Rails

I started in the HOME folder and only moved when it was time to clone Tracks.

Update your Pi as needed using the GUI then update package handler. This took some minutes:
   1. `sudo apt update`
   2. `sudo apt full-upgrade`
   
   
  Then I used these instructions to install ruby [Installing Ruby Debian](https://www.ruby-lang.org/en/documentation/installation/#apt):   
1. `sudo apt-get install ruby-full`
2. `sudo apt-get install ruby-dev`

It takes some minutes to finish.

Rails also needs **SQLite3** according to [Getting Started with Rails](https://guides.rubyonrails.org/getting_started.html#creating-a-new-rails-project-installing-rails)
1. `sudo apt install sqlite3`
2. `sqlite3 --version`
I got 3.4 as the version.

Warm up your train because it is finally time to install Rails. In my first tries I used sudo but later figured out how to install without it. Here I temporarily add the gems path to the user path:
1. `gem install rails --user-install`
2. `PATH="$HOME/.local/share/gem/ruby/3.1.0/bin:$PATH"`
3. `rails --version`

This took some minutes but I got 7.2.1 as the version.

To **permanently** add the path most people suggest to add this path to the end of .profile, a hidden file in your user home folder. The easiest way is to use this command:
1. `echo "PATH="$HOME/.local/share/gem/ruby/3.1.0/bin:$PATH" >> .profile`

![end of .profile](./notepad_wMF08dC0Yr.png)


That was the end of the good times :-(
In my first attempts I tried **bundle config set without mysql postgresql** & **bundle install** but it kept getting stuck. This was after I installed bundler by itself. I actually let it ran overnight just to be sure for gems like puma, tracks_chartjs_ror and finally libv8-node.

The next 2 subsections help with the above issues. 

### Installing nodejs & yarn

Getting stuck installing the gems above and later getting the error message that it couldn't find yarn finally led me to the path of abandoning **mini_racer** and *therubyracer* and installing *nodejs* as a Linux package. Used the instructions from [Node.js and Raspberry Pi](https://www.w3schools.com/nodejs/nodejs_raspberrypi.asp) and here for [yarn](https://classic.yarnpkg.com/lang/en/docs/install/#windows-stable). This would be my first but not the last deviation from the official instructions.

By the way, trying to install yarn with apt-get would lead to **cmdtest** being installed. I might have purged it as some point before the following commands with sudo apt-get purge cmdtest.

Running the following commands took some time:
1. `curl -sL https://deb.nodesource.com/setup_20.x | sudo bash -` Here I used the [recommended version 20.](https://deb.nodesource.com/-->20)
2. `sudo apt install nodejs -y`
3. `sudo npm install -g yarn` Couldn't easily install without sudo
4. `yarn global bin`

The yarn folder is populated with yarn v1.22.22 and that is what command 4 should output.

![Installing yarn.](./C97qgG0s1l-1.png)

### Installing individual gems

In this subsection I will share what gems I installed individually. This would be my second deviation from the official instructions. I don't know why, but it kept getting stuck and after trial and error I found a way to make it work. My suspicion is that there is nothing wrong with the Gemfile itself, but installing individually helps the system from trying to do too much and crashing.

It took time but like with rails I also figured how to install without sudo. [Here](https://www.moncefbelyamani.com/why-you-should-never-use-sudo-to-install-ruby-gems/) and [here](https://felipec.wordpress.com/2022/08/25/ruby-stop-using-sudo/) are some good reasons why one should avoid sudo.

Install the following gems. Some will take seconds, others minutes:

1. `gem install bundler --user-install`
2. `gem install puma --user-install`
3. `gem install chartjs-ror --user-install`
4. `gem install tracks-chartjs-ror --user-install`
5.  `gem install listen --user-install --verbose`  It takes 5 minutes, used verbose as sanity check.
6.  `gem install bootstrap --user-install` 2 minutes no need for verbose
7.  `gem install sassc --user-install --verbose` It took a long time but finished.
8.  `gem install sassc-rails --user-install` Took seconds.
9.  `gem install bootstrap-sass -v 3.4.1 --user-install --verbose` Takes a few minutes.

![Here is a screenshot](./6gHwO8nbvI.png)


### Cloning the Tracks repository

Now, lets return to the official instructions and install Tracks in Sites/tracks:
1. `mkdir Sites`or another folder that you will use to clone Tracks into.
2. `cd ~/Sites` 
3. `git clone https://github.com/TracksApp/tracks.git`
4. `cd tracks`

I tried with and without Github, it was far easier using it. Plus with git there isn't that menacing `Fatal Error` message.

![Screenshot of git process](./QITpMCHMZL.png)


### Editing Gemfile

To be laconic, one gem is in the wrong place in the Gemfile.

Omitting this fix will give you the dreaded Sprocket error message when trying to precompile.

#### ~~sassc-rails~~

**Ignore this whole part and proceed to bootstrap-sass.** 

Although habernal was right about it taking a long time, it doesn't freeze if sassc was already installed as per my above instructions.
I am leaving it in this guide because it might help someone
```
~~Many thanks to **habernal** for figuring out the [sassc-ruby issue](https://github.com/TracksApp/tracks/issues/2693).
The solution is to edit the Gemfile and Gemfile.lock to have:
1. In Gemfile move `sassc-rails` line from **group :assets do** upwards just below `gem 'will_paginate'`.
2. Change the version to 2.1.0 so that the line is now `gem 'sassc-rails', '~> 2.1.0'` .
2. In Gemfile.lock near the end or line 378 change the version so that the line in the **DEPENDENCIES** section becomes `sassc-rails (~> 2.1.0)`.
3. I only changed the version in **DEPENDENCIES**.~~

```


#### bootstrap-sass

However you **must** follow these instructions. I found the solution in stackoverflow, many thanks to that one person :-)

In the Gemfile:
1. Move `gem 'bootstrap-sass', '3.4.1'` line from **group :assets do** upwards so it should look something like this:

```
gem 'tracks-chartjs-ror'
gem 'will_paginate'
gem 'bootstrap-sass', '3.4.1'
```
Save the file and proceed to the next subsection.

### Tracks installation

With the Gemfile saved, return to the official instructions to config and use bundle for the installation:
1. `bundle config set without mysql postgresql therubyracer` No output.
2. `bundle config path /home/fc/.local/share/gem/ruby/3.1.0/bin` No output, required because of --user-install while installing gems. 
3. `bundle install` After it starts turn off your screen and got get a beverage. This can take 30 minutes to 2 hours.

Only the third command will have an output.

![Only the third command will have an output.](./g5Ghisxyd3.png)

If everything worked it will install some gems and end with a nice green message. Maybe a warning about S3 storage too.

![Only the third command will have an output.](./6ez9Ahvf8t.png)

Following the official instructions it is time to [Configure variables](https://github.com/TracksApp/tracks/blob/master/doc/installation.md#configure-variables):
1. `cd config`
2. `cp database.yml.tmpl database.yml`
3. `cp site.yml.tmpl site.yml`
4. `geany database.yml or use another editor to open it`
5. Delete/comment out all other production sections except the last section and change that production example to `adapter: sqlite3` and `database: db/tracks.db`. I posted mine below.
6. Save and exit
7. `geany site.yml or use another editor to open it`

```
production:
  adapter: sqlite3
  database: db/tracks.db
  pool: 5
  timeout: 5000
```
I recommend opening another terminal, navigate to the root folder of tracks (Sites/tracks) to run the following commands:
1. `bundle exec rake time:zones:local`
2. `rake secret`

I found it easier to send the output in a file. By the way in the following screenshot, visible before `cd config` is the last output from `bundle install`.

![ for easy copying at a convenient time.](./eGBAVnSCCi.png)

Returning to the editor you will need to change the following lines:
1. `time_zone: "UTC"` Replace with one of the results of 1 above.
2. `secret_token: " "` . Paste the token generated by command 2 above.
3. Set `open_signups: true`
4. Add an admin email if you want.
5. I will experiment with the other settings in the future.
6. Save and exit.
7. Return to the tracks root folder by `cd ..` or just close this terminal and use the other one.

Now lets [populate your database with the Tracks schema](https://github.com/TracksApp/tracks/blob/master/doc/installation.md#populate-your-database-with-the-tracks-schema):
1. `bundle exec rake db:migrate RAILS_ENV=production`

 If you didn't use Github it will complain with a **fatal: not a git repository** now and for the next command but ignore this. There is a lot of output from this command, but usually finishes in a minute.
 
 ![populate your database with the Tracks schema](./ePbRVw6gVX.png)


Lastly lets [precompile assets](https://github.com/TracksApp/tracks/blob/master/doc/installation.md#precompile-assets):
1. `sudo bundle exec rake assets:precompile RAILS_ENV=production`

If you skipped the previous subsections you will also get other error messages about:
1. yarn
2. Sprocket **Can't run bootsrap : couldn't find file 'bootstrap-sprockets' with type 'application/javascript'**  and the list a lot of folders that it looked for that file. The solution was primarily to edit the Gemfile and move gem 'bootstrap-sass', '3.4.1' with this exact version number. Installing the gems individually may have helped. I spent a lot of time figuring this out but luckily stackoverflow helped.

Success means that it will start precompiling lots of files that will be listed on your screen. Give it a few minutes.


![start precompiling lots of files](./WMgwB5PgOz.png)

Example of not using git to get Tracks below:
```
fatal: not a git repository (or any of the parent directories): .git
fatal: not a git repository (or any of the parent directories): .git
yarn install v1.22.22
[1/4] Resolving packages...
success Already up-to-date.
Done in 0.21s.
I, [2024-09-19T23:41:37.037734 #2996]  INFO -- : Writing
```


### Starting the server

Finally the moment of truth! I faced a few issues here to:
1. Firstly the command given in the [Start the server](https://github.com/TracksApp/tracks/blob/master/doc/installation.md#start-the-server) instructions wouldn't work.
2. Then when the server finally started, I was able to access it and signup, but it had no graphics.

I finally realized I needed to be using the executables in the bin folder. Maybe this is because of that sudo bundle command. But examining the site in my browser following 2 above I realized it couldn't find assets even thought they were there. So I put my combinatronics in good use and tested multiple combinations of that command. The successful commands varied a bit but if you have been following this guide the **first** one should work for you too. Just make sure you are in the tracks root folder

Here is what worked:
1. `RAILS_SERVE_STATIC_FILES=TRUE bundle exec ./bin/rails server -e production`
2. `sudo RAILS_SERVE_STATIC_FILES=TRUE bundle exec rails server -e production`
3. `RAILS_SERVE_STATIC_FILES=TRUE ./bin/rails server -e production`


A glorious message that your server has started will appear.

![glorious message](./YruvvkctQv.png)

After this you can use the Pi or your daily driver to access the site. Write down the Pi IP in your local network something similar to 192.168.xxx.123.

From the Pi head over to http://localhost:3000/signup or http://0.0.0.0:3000/signup to make an admin account and the logout to make a user account. 

From another computer in your network you can access the site at the Pis IP:3000 similar to this example http://192.168.xxx.123:3000.

![trackslogin](./TRACKS Login.png)


### Adding to autostart
And of course I would like Tracks to start every time I boot maybe using the user method from [How to use Autostart - Raspberry Pi OS (Desktop).](https://forums.raspberrypi.com/viewtopic.php?t=294014&sid=a8f7a3a612ceb11842d9394cfb7ab39f)

Starting from $HOME
1. `cd .config`
2. `mkdir lxsession`
3. `mkdir lxsession/LXDE-pi`
4. `cp /etc/xdg/lxsession/LXDE-pi/autostart lxsession/LXDE-pi/`
5. `cd lxsession/LXDE-pi/`

The original autostart contents:
```
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi
@xscreensaver -no-splash
```
Append to the file the to command start a Bash script (autostarttracks.sh in /home/fc) with terminal:
1. `echo "@lxterminal -e bash /home/fc/autostarttracks.sh" >> autostart`

#### The script

Keep in mind that the script (autostarttracks.sh) has to be Unix file and an executable for the owner
An easy way is to use the command line in the path defined above:
1. `echo "#!/bin/bash" > autostarttracks.sh`
2. `echo " " >> autostarttracks.sh`
3. `echo "cd Sites/tracks" >> autostarttracks.sh`
4. `echo "RAILS_SERVE_STATIC_FILES=TRUE bundle exec ./bin/rails server -e production" >> autostarttracks.sh`
5. `chmod +x autostarttracks.sh`


Here is the final result for this subsection:

![ final result for this subsection](./powershell_pYiGz4IfXL.png)



---

## 4. The Dream

I don't use [GTD](https://gettingthingsdone.com/) for everything but it is a method that is very close to the way I do things. Tracks is one of the nicest free solutions I have ever seen after two decades in the productivity rabbit hole.

Ultimately I want to end up with a solution that allows me to elegantly host in my local network and activate:
1. the SSL features
2. the email alerts

As for physical placement, after looking online for an elegant solution I only found this [solution posted in Medium by Jamie Bailey ](https://medium.com/initial-state/how-to-build-a-plug-in-pi-zero-display-and-show-something-useful-fd402c912a22). And it is awesome! I now have my own server with tracks!

Fun fact: You can install this in one Pi and then drop it in your Zero and it will still work! You may have to save the connection again.

I hope you find this guide useful and I look forward to your feedback. May your Tracks be full and your mind clear!
