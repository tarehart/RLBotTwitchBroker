# RLBotTwitchBroker

RLBotTwitchBroker is a system for allowing users on twitch chat to have influence on bots and scripts in a RLBot Rocket League game. 

Example scenario:
- A human is playing against a bot, and there's also a script active that can change boost levels in cars.
- TwitchBroker queries the script and the bot asking what actions are possible, and shows the options in an overlay.
- A user in twitch chat enters some command, e.g. `!rlbot 5` to select one of them which might say "5: Drain boost from human".
- TwitchBroker reads the command and tells the script to drain the boost.

This can also be used to influence the strategy of bots, e.g. forcing them to take shots, rotate out, etc. The system is designed so that people can easily update their existing bots to support the action interface.

## Getting started

1. Clone the repository.
1. Install all 5 packages to whatever pip repository you intend to use.
   - If it's your system python, you can run install_packages_locally.bat to do this easily.
   - If you wish to use RLBotGUI, installing the packages will *not* work if it's using the bundled python instance. You can use an installation technique like https://github.com/RLBot/RLBotGUI/tree/install/alternative-install to make it use a full python instance.
1. Start a RLBot match and get all the scripts in /tst running at once
   - This can be done by loading /tst in RLBotGUI and toggling on the scripts that appear.
   - You can also just launch the scripts manually.
1. At time of writing, the twitch_broker.py script will now present you with options to choose via the command line, which give you control over the boost_monkey and the included bot.

## Design

![diagram](https://i.imgur.com/IbzEBnq.png)

![diagram](https://i.imgur.com/klspJS7.png)

We've used [Swagger](https://swagger.io/) to define a rest interface so that TwitchBroker can communicate with scripts and bots.

## Code Tour

Look in /tst to see what user code is expected to be like.
  - boost_monkey is a *script*, not a bot, which can manipulate the boost levels of any car in the game.
  - bot is a bot that can be told to steer left or right.
  - twitch_broker is a trivial launcher script that calls into the rlbot_twitch_broker library.
  
To make this work, we have five packages:
  - rlbot_action_server: Bots and scripts will run this server so they can share what actions are available and accept commands.
  - rlbot_action_client: The rlbot_twitch_broker will use this to talk to rlbot_action_server instances
  - rlbot_twitch_broker_server: The rlbot_twitch_broker will run this server so it can listen for "registrations" from bots and scripts in the current match that want to participate in the system.
  - rlbot_twitch_broker_client: Bots and scripts will use this to talk to rlbot_twitch_broker_server to register themselves.
  - rlbot_twitch_broker: Reads available commands from bots and scripts, produces an overlay, reads commands from twitch, generally makes everything happen.
  
There's a lot of code, but a large portion of it is auto-generated by swagger. This is done by executing `generate-action-swagger.bat` and `generate-broker-swagger.bat`. It's still save to run these, because I have used .swagger-codegen-ignore to protect those files which we've customized from being overwritten. At the moment, the `_client` packages are purely generated with zero customization, and the `_server` packages have small tweaks to allow the controllers to be implemented by outside code.
