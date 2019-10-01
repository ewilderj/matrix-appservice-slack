# Getting Started

In this guide we will setup the bridge connected to your homeserver. This guide
will walk through the most common settings, other options are documented in
other places.

## Installation

### From source

```sh
$ git clone https://github.com/matrix-org/matrix-appservice-slack.git
$ cd matrix-appservice-slack
$ npm install
$ npm run build
```

### With Docker

```sh
$ docker pull matrixdotorg/matrix-appservice-slack:latest
```

## How it Works:

The bridge listens to events using the Slack RTM API over websockets, and to
matrix events on a port that the homeserver sends events to. This tutorial will
walk you through configuring your homeserver and Slack to send messages to this
bridge and setting up the api so this bridge can relay those message (all
messages) to the other bridged channel. For the sake of this tutorial, we will
assume your homeserver is hosted on the same server as this bridge at the port
`8008` (http://localhost:8008).

If you've set up other bridges, you're probably familiar with the link used
to reach your homeserver, the "homeserver url". This is the same URL. This
is the same port. No problem! Multiple bridges can plug into the same
homeserver url without conflicting with each other.

NOTE: If your bridge and homeserver run on different machines, you will need
to introduce proxying into the mix, which is beyond the scope of this readme.
There are some really awesome and kind people in the Matrix community. If you're
ever stuck, you can post a question in the [Matrix Bridging channel]
(https://matrix.to/#/#bridges:matrix.org).


## Setup

1. Create a new Matrix room to act as the administration control room. Note its
   internal room ID (EX: !abcdefg12345hijk:coolserver.com).

2. Decide on a spare local TCP port number to use. It will listen for messages
   from Matrix and needs to be visible to the homeserver. Take care to configure
   firewalls appropriately. This ports will be notated as `$MATRIX_PORT` and in
   the remaining instructions.

3. Create a `config.yaml` file for global configuration. There is a sample
   one to begin with in `config/config.sample.yaml` you may wish to copy and
   edit as appropriate. The required and optional values are flagged in the config.

4. See [datastores](docs/datastores.md) on how to setup a database with the bridge.

5. Generate the appservice registration file. This will be used by the
   Matrix homeserver. Here, you must specify the direct link the
   **Matrix Homserver** can use to access the bridge, including the Matrix
   port it will send messages through (if this bridge runs on the same
   machine you can use `localhost` as the `$HOST` name):
   
   ```
   $ npm start -- -r -c config.yaml -u "http://$HOST:$MATRIX_PORT"
   ```
   
   or with docker:
   
   ```
   $ docker run -v /path/to/config/:/config/ matrixdotorg/matrix-appservice-slack -r -c /config/config.yaml -u "http://$HOST:$MATRIX_PORT" -f /config/slack.yaml
   ```

6. Start the actual application service.

   ```
   $ npm start -c config.yaml -p $MATRIX_PORT
   ```
   
   or with docker:
   ```
   $ docker run -v /path/to/config/:/config/ matrixdotorg/matrix-appservice-slack
   ```

7. Copy the newly-generated `slack-registration.yaml` file to your Matrix
   homeserver. Add the registration file to your homeserver config (default
   `homeserver.yaml`):

   ```
   app_service_config_files:
      - ...
      - "/path/to/Slack-registration.yaml"
   ```

   Don't forget - it has to be a YAML list of strings, not just a single string.

   Restart your homeserver to have it reread the config file an establish a
   connection to the bridge.

8. Invite the bridge bot user into the admin room, so it can actually see and
   respond to commands. The bot's user ID is formed from the `sender_localpart`
   field of the registration file, and the homeserver's domain name. For example:

   ```
   /invite @slackbot:my.server.here
   ```

NOTE: At the time of writing, Riot does not recognize the Slack bot. This is
okay. The bot *is there*... probably. Either way, when Riot asks if you're
sure you want to invite @slackbot, just say yes.

The bridge bot will stay offline for most of the time. This is normal. You
will know if the bridge is working (and that your homeserver is properly
connected) if it accepts your invitation. You can expect the bot to accept
within 45 seconds of being invited. If it never accepts the invitation,
check your bridge's logs and review the above steps.

The bridge itself should now be running. Congrats!

To actually use it, you will need to configure some linked channels, see
[linking channels](link_channels.md).