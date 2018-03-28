Greetings,

so you want to install the angband-webclient on your Raspberry Pi.
Here are my notes on that journey from March 2017, your mileage may vary.
As an aside, trying this on Windows was a complete disaster.

**Update all of the Raspbian packages First**

This should be done from time to time anyway

`sudo apt-get update`

`sudo apt-get upgrade`

If you still run into trouble, you can try to upgrade the whole distribution

`sudo apt-get update`

`sudo apt-get dist-upgrade`

`sudo apt-get install -y pprompt`


**Turn on SSH (optional)**

You most likely will not want to do all your coding on the Raspberry,
you can connect to your Raspberry via SSH, but you need to turn it on.
This can be done in the console wit the command

`raspi-config`

Assuming you are logged in with the pi user, change the default password
from `raspberry` to something else with the command

`passwd`

**Install node.js and NPM**

nodejs should already be installed, but there is a good chance that you
will find npm missing. If so, run this:

`curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -`

`sudo apt-get install -y nodejs`

**Install mongodb**

This is done with this command:

`sudo apt-get install mongodb-server`

Running the server calls for 

`sudo service mongod start`

In my case, it did not work immediately, because the `/data/db` folder
did not exist. I create the folders with `sudo mkdir` and then had to
use `chown` to change the owner back to `pi`. (pi is not allowd to 
create folders in the root)

From there I had to run

`sudo rm /var/lib/mongodb/mongod.lock`

`mongod --repair`

`sudo service mongodb start`


**Changes to Hellband**

A separate makefile was made, highlights are
* Do no use private user paths, it will not work
* Set `DEFAULT_PATH` since binaries and the `lib` folder are not expected in the same place
* If any path uses `~`, then the `stat` function will not work on that path

**Changes to angband-client**

I hope Gwarl will accept some patches, but for now:
* Change `lib.js`
  * Do not console.log(args), but console.log(termdesc), this saves so much troubleshooting time
* Change `app.js`
  * There is a hardcoded reference to `/home/angbandlive` which should be adjusted
      * Add `var home          = process.env.CUSTOM_HOME || '/home/angbandlive';`
      * Go for `app.use(express.static(home + '/public/user'));`
  
**Setting CUSTOM_HOME**

The code by default will have everything point `/home/angbandlive`.
This is most likely not relevant to you, so set CUSTOM_HOME to a relevant location with `export`

**Creating a service**

From https://thomashunter.name/blog/running-a-node-js-process-on-debian-as-an-init-d-service/
`sudo npm install -g forever`
`sudo mkdir /var/run/forever`

Place this in etc/init.d (I called it hellband-webclient)

    #!/bin/sh

    export CUSTOM_HOME=~/hellband-live

    case "$1" in
      start)
      exec forever --sourceDir=<wherever your reps are>/angband-webclient/ -p /var/run/forever start ./bin/www
      ;;

      stop)
      exec forever stop --sourceDir=<wherever your reps are>/angband-webclient/ ./bin/www
      ;;
    esac

    exit 0
    
 Then run the following the file (I replaced SERVICE with hellband-webclient)
 
     sudo touch /etc/init.d/SERVICE
     sudo chmod a+x /etc/init.d/SERVICE
     sudo update-rc.d SERVICE defaults

Then run sudo service SERVICE start
