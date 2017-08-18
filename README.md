# Broadlink RM Server for IFTTT control

This is a simple server that allows you to connect your Broadlink (tested with the RM mini 3) to your IFTTT account.

# Setup
- Get a IFTTT account and enable the Maker Webhooks: https://ifttt.com/maker_webhooks
- Download and install ngrok: https://ngrok.com/
- Make sure that you have an up-to-date version of node with ```npm install npm@latest -g``` (https://docs.npmjs.com/getting-started/installing-node)
- Install this repository dependencies by running ```npm install```
- Install the Broadlink app and setup your home Wifi (I asume your Broadlink is connected to the same network as your server from now on)
- Download the RM Bridge app (Android only). This is needed to learn the IR codes for your appliances, but won't be needed beyond then.

# Start the server
After installing the dependencies, run ```npm start``` to initialize the server. You will see in the console your Broadlink IP address, we will use it later.  You'll need to restart the server (CTRL+C, then rerun ```npm start```) each time you edit commands.js (next section).

# Learn some IR codes
Open the RM Bridge app and start the service, then, navigate to: http://rm-bridge.fun2code.de/rm_manage/code_learning.html

Configure the website with your RM Bridge IP and PORT and click Load devices. If you can't find your device you will need to manually add it using the *Add manually* option. Google how to get the MAC Address from the IP you got when running the server.

Once added, click on Learn Code and point your remote to the Broadlink, press the button you want to learn. You will get a JSON with the details to run this code, copy the *data* value (this is a HEX of the IR code you just sent to the Broadlink).

Modify the *commands.js* file with this code (follow the instructions there).  Note that the input command names will have spaces removed and converted to uppercase, so make sure you do the same with your names.  

To create commands that consist of multiple codes (e.g., to have your TV switch to a specific input), simply chain together the hex data from the indivdual button presses in a list.  You may have to do some trial and error to get the right hex code for a *single* button press.  For example, I was having trouble selecting the right option in my input list because a single hex string was sending two ```down arrow``` commands.

## Ngrok
In order to connect IFTTT to the PC/server running this code (like a Raspberry Pi), you will need a URL that tunnels to your device, this is done with ngrok. 

After installing ngrok, run: ```ngrok http 1880```

## IFTTT
For this instructions I'm going to setup my Google Home with the Broadlink, on IFTTT search for Google Assistant and enable it (https://ifttt.com/google_assistant). Then, create a new applet: https://ifttt.com/create.  There are then two ways that you can take this -- create a single applet for each command, or create one over-arching applet that handles all of the commands.  Each has it's benefits:  the former allows you to more readily customize how you trigger the command, while the latter makes it easier to update your IFTTT settings if you ever have to restart ngrok (and therefore get a new http address).

### One applet per command:

#### Trigger setup
- Click on *+this* and search for the Google Assistant.
- Select *Say a simple phrase*
- Set *What do you want to say?* with a command like "TV on".
- Set *What do you want the Assistant to say in response?* with a custom response like "Turning the TV on"
- Click on *Create trigger*

#### Action setup
- Click on *+that* and search for Webhooks
- Select *Make a web request*
- In the URL field, set your Ngrok URL and add at the end ```/command/YOUR_COMMAND_NAME```
- Method should be POST
- Content Type select *application/json*
- For the body set: ```{"secret":"YOUR_COMMAND_SECRET_HERE"}```
- Click *Create Action*

You should now be able to say *Ok Google, TV ON* and IFTTT will send a request to your local server that will relay the action to the Broadlink and turn on your TV!.  Repeat this (i.e., create a new applet) for each command that you want to run.

### One applet to do all commands:

In order for this to work, your ```YOUR_COMMAND_SECRET``` must be the same across all commands.

### Trigger setup:
- Click on *+this* and search for the Google Assistant.
- Select *Say a simple phrase with a text ingredient*
- Set *What do you want to say?* with a trigger word/command like "remote" or "send", followed by a dollar sign (i.e., "remote $").  The "$" represents the text ingredient.
- Set *What do you want the Assistant to say in response?* with a custom response like "Sending $"
- Click on *Create trigger*

#### Action setup
- Click on *+that* and search for Webhooks
- Select *Make a web request*
- In the URL field, set your Ngrok URL and add at the end ```/command/``` and then click "Add Ingredient" and select TextField.  Make sure there are no spaces.
- Method should be POST
- Content Type select *application/json*
- For the body set: ```{"secret":"YOUR_COMMAND_SECRET_HERE"}```.  As noted above, your secret must be the same for all commands.
- Click *Create Action*

# Credits:
Some parts of the code are from @lprhodes Homebridge Broadlink:
https://github.com/lprhodes/homebridge-broadlink-rm

Also his module for the communication is used.
