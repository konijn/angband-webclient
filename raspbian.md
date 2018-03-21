Greetings,

so you want to install the angband-webclient on your Raspberry Pi.
Here are my notes on that journey from March 2017, your mileage may vary.

** Update all of the Raspbian packages First **
This should be done from time to time anyway
`sudo apt-get update`
`sudo apt-get upgrade`

If you still run into trouble, you can try to upgrade the whole distribution
`sudo apt-get update`
`sudo apt-get dist-upgrade`
`sudo apt-get install -y pprompt`

** Turn on SSH (optional) **
You most likely will not want to do all your coding on the Raspberry,
you can connect to your Raspberry via SSH, but you need to turn it on.
This can be done in the console wit the command
`raspi-config`
Assuming you are logged in with the pi user, change the default password
from `raspberry` to something else with the command
`passwd`

** Install node.js and NPM **
nodejs should already be installed, but there is a good chance that you
will find npm missing. If so, run this:

`curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -`
`sudo apt-get install -y nodejs`

** Install mongodb **
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




