# ValueBot

A Slack bot for calling out coworkers for espousing your organization's values. The best way to use ValueBot is have it listen across all the channels of your team. That way, anyone can call out other team members without disrupting their workflow. If you prefer, it can also listen only on a channel you specify, for instance `#values` or `#valuebot`.

## Usage

A call-out is a message that includes a username and a hashtag representing a value to call the user out for. You provide the list of hashtags to listen to in the config file (described below). For example, if I wanted to call out `@mary` for an innovative solution to a problem that's been bugging the team for a while, that message would look like:

```
#innovative solution, @mary!
```

### Posting

To get ValueBot to hear you, you need to start your message with either `valuebot` or with the hashtag you're using. Using the example above:

```
valuebot @mary that was one #innovative solution!
```

or

```
#innovative @mary kudos for solving that problem!
```

You can post a message in either format on any channel that ValueBot is listening on, and it'll store the message in its database.

### Getting Lists

Admins can request lists from ValueBot of posts about a certain user, of posts with a certain value, and of the users with the most posts about them. You define which users are admins in `options.py`, as described below.

Regular users can also ask ValueBot for lists, but are limited to only asking for a list of posts about themselves, which is done by passing in 'me' when asking for posts by user.

#### By User

To get all the posts about a certain user, post a message to any channel that ValueBot is listening in with the following format:

```
valuebot list [user] [time]
```

- `user` should be the username of someone in your team.
  - Can also be 'me' to request posts about yourself (this command is also available to non-admin users).
- `time` (optional). If no `time` is specified, it defaults to getting posts from all time.
  - 'today' to get the posts today.
  - `month [year]` to get the posts from that month. If no year is provided, it's assumed to be the current year.

#### By Value

To get all the posts with a certain value, post a message to any channel with the following format:

```
valuebot list [value] [time]
```

- `value` should be either:
  - 'all' to get posts across all values.
  - The value you want to search for, as defined in your `options.py` file. Can also be a hashtag associated with that value.
- `time` (optional) in the same format as above.

### Leaders

To get the users with the most posts, post a message with the following format:

```
valuebot list leaders [value] [time]
```

- `value` in the same format as above, to specify for which value ValueBot should find leaders.
- `time` (optional) in the same format as above

## Setting it up

You can set up ValueBot on your own server/Slack team in three steps: create a config file, create the database, and set up Slack's Incoming/Outgoing WebHooks.

### Config file

To start, create a file named config.py in the root of this directory. The four options you need are `SQLALCHEMY_DATABASE_URL`, `WEBHOOK_URL`, `ADMINS`, and `HASHTAGS`.

```
# config.py

SQLALCHEMY_DATABASE_URI = "sqlite:///db/valuebot.sqlite3" # Or wherever you want the database to live

WEBHOOK_URL = "YOUR_URL_HERE"   # The URL configured on Slack for Incoming Webhooks
SLACK_TOKEN = "YOUR_TOKEN_HERE" # The token for the Slack API you create in the next step

ADMINS = {"admin", "admin2"}    # A set of usernames corresponding to the users in
                                # your team to whom you want to give admin privileges.

HASHTAGS = {                    # A dictionary with keys of values you want to be able
  'innovation': {               # to call people out for, corresponding to sets of
    '#innovation',              # hashtags that users can use to refer to those values.
    '#innovative',
    '#be-innovative' 
  },
  'passion': {
    '#passion',
    '#passionate',
    '#be-passionate'
  }
}
```

You can use the `SQLALCHEMY_DATABASE_URI` value provided above as-is, in which case the data will be stored in `db/valuebot.sqlite3` in this directory. Alternatively, you can provide a URI to another SQL database, which can be with SQLite, MySQL, PostgreSQL, or [any database supported by SQLAlchemy](http://docs.sqlalchemy.org/en/rel_1_0/dialects/index.html).

To find your `WEBHOOK_URL`, you need to add ValueBot to your Slack team's integrations on the Slack web interface, which you can do in the next step.

### Database

To get the database set up, first make sure that you've specified `SQLALCHEMY_DATABASE_URI` correctly in `config.py`. Then, simply run the following command from the project directory to set up the database schema:

```
$ python app.py db upgrade
```

### Slack

To integrate ValueBot with Slack, you need to set up Outgoing Webhooks, Incoming Webhooks, and the Slack API.

#### Outgoing Webhooks

Outgoing Webhooks are used by ValueBot to "listen" to messages sent across an organization. To set it up, create a new Outoing Webhooks Integration at `https://[your team].slack.com/services/new`, and use the following settings:

![Outgoing Settings 1](http://i.imgur.com/MCMsiNH.png)

To get your trigger words, you can run the included helper function like so:

```
$ python app.py trigger_list
```

Simply paste the produced string into the input for "Trigger Word(s)." For the URL field, include the URL of the server where you're running ValueBot.

For the cosmetic settings, it's recommended to identify ValueBot with a name and icon, although it's not strictly necessary.

![Outgoing Settings 2](http://i.imgur.com/CfBoyyq.png)

#### Incoming Webhooks

Incoming Webhooks are used by ValueBot to send private messages of the generated lists to users. Ideally, we could use only Outgoing Webhooks to accomplish this, but as of ValueBot's creation, Slack's API does not support sending private messages with only Outgoing Webhooks. Go [bug Slack about it](https://api.slack.com/), if you'd prefer a simpler installation process.

To configure Incoming Webhooks, create a new Incoming Webhooks ingegration at `https://[your team].slack.com/services/new`. The settings here are entirely optional, as ValueBot won't actually post to the channel you specify. The most important thing is the Webhook URL, which Slack generates for you. Take this url, and put it into your `config.py` as `WEBHOOK_URL`. Now, ValueBot knows how to send private messages to your organization.

![Incoming Settings](http://i.imgur.com/rz3KPrQ.png)

#### Slack API

ValueBot uses the Slack API to figure out what users are being mentioned in call-out posts. It's not ideal, but this is essentially necessary to get around the issue [outlined in this Gist](https://gist.github.com/ernesto-jimenez/11276103), in which Outgoing webhooks doesn't include username information. Once again, this isn't ideal, and you can go [bug Slack about it](https://api.slack.com/) to get this feature added to Outgoing Webhooks :)

To configure ValueBot to get around this, you simply need to generate a Slack Web API token on the [API homepage here](https://api.slack.com/web):

![Slack Web API](http://i.imgur.com/VAbt9UQ.png)

Once you have this token, put it into your `config.py` as `SLACK_TOKEN` and you're good to go!

## Running the Server

Once you've completed the set up, all that remains is to run the server that backs ValueBot. To do this, simply run the command:

```
$ python app.py runserver [-p port]
```

Where `port` is an optional argument for which port the server should run on. Defaults to 4567. Once you have the server running, and you've hooked up Slack Webhooks, you're good to go! Now you can start calling out your co-workers, and keeping track of who's doing the best job espousing your organization's values.

## Extra

ValueBot also comes with a convenient function to send a list of yesterday's value leaders to your slack organization. The command to do this is:

```
$ python app.py send_yesterday_leaders [channel]
```

Where `channel` is any channel in your Slack organization. For instance, you could specify `"#general"` to send an update to the whole organization about who led the team in call-outs the previous day. You can also hook this into a `cron` job to automate sending this message every morning, for instance.
