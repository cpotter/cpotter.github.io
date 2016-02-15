---
title: Log4js Appender
published_at: 2016-2-14 9:47 PM
author: Chris Potter
image: /2016/2/14/images/log4js-node.png
---
To review here is the list of things that I claimed I would work on:

 - [Ghost/Github Blog](https://github.com/cpotter/cpotter.github.io)
 - Deck of Cards API vanilla node app
 - Record Cataloger Meteor App
 - Meteor 1000 Books by Kindergarten Meteor App
 - Sudoku Solver
 - Freeflow Solver
 - Crafty Spending App with [Charlestronauts](https://github.com/Charlestronauts)

 So the big question is...
### What have I been working on?
And the winner is, none of the above!  At my job at [Blue Acorn](https://github.com/BlueAcornInc), I have had the priveledge of working on some side projects with [@BriceBurgess](https://github.com/briceburg), our resident Dev-Ops aficionado. Dev-Ops has been a personal goal of mine over the last year to contribute more to [Blue Acorn](https://github.com/BlueAcornInc). One of our major company initiatives has been to coordinate the automation process for building better websites, with continuous integration testing which will result in more robust deployments.  It has been great fun with I am learning a lot.

#### How does this relate to log4js-node?
One of the resources that we use at [Blue Acorn](https://github.com/BlueAcornInc) is the Atlassian product, [Hipchat](www.hipchat.com).  Hipchat is slowly being adopted as our company chatting tool, but also as a source of logging and recording for our different Dev-Ops tools.  For our un-hipchat-integrated internal apps, we have been using the [log4js-node](https://github.com/nomiddlename/log4js-node) library, which is the node version os [log4js](http://stritti.github.io/log4js/docu/users-guide.html), which allows a user to set log levels for better logging information for your application to log to your console.  However, another feature of the log4js library is the concept of Appenders.

#### Finally just tell me what the point is
I decided to extend the log4js library to support logging to the hipchat client, using both [log4js-node](https://github.com/nomiddlename/log4js-node) and a new node library [hipchat-client](https://github.com/germanrcuriel/hipchat-client) with an appender specific to hipchat.  Hipchat-client implements the hipchat V1 api to connect to hipchat, allowing you to post messages or notifications to specific rooms.  Thankfully there were a number of examples provided with the log4js-node github repo but the most useful resource I was able to use was the [mailgun.js](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/lib/appenders/mailgun.js), which functions to allow lof4js to send it's log through a registered mailgun account.

##### Lets look at [hipchat.js](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/lib/appenders/hipchat.js)
The file is relatively straight forward.  Every log4js appender needs to have 2 distinct functions that the log4js-node library looks for:
```
exports.name = 'hipchat';
exports.configure = function(){[...]};
exports.appender = function(){[...]};
```

These functions inform log4js how to configure the new appender, or logging service, within the log4js library.  An [example](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/examples/hipchat-appender.js) of how to use the hipchat appender might better inform how these functions work:
```
var log4js = require('log4js-node');

log4js.configure({
  "appenders": [
    {
      "type" : "hipchat",
      "api_key": 'Hipchat_API_V1_Key',
      "room_id": "Room_ID",
      "from": "Tester",
      "format": "text",
      "notify": "NOTIFY",
      "category" : "hipchat"
    }
  ]
});

var logger = log4js.getLogger("hipchat");
logger.warn("Test Warn message");//yellow
```
When configuring your log4js instance within your application, when you declare your appenders in the json, the log4js module will loop over all the appenders listed in the `appenders: [ ]` and load them into the library instance by their type, which should match the `exports.name` provided in the appender, [found here](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/lib/log4js.js#L221). It will bind the above configuration to the appenders `exports.configure` method based off of the category provided.  Then when `.getLogger('hipchat')` is called, it will load an instance of the saved appender with the bound configuration provided from the json object. When the form of log is called, `logger.warn('Test Warn Message')` or `logger.error("Test Error Message");` this will call the `exports.appender` method.
#### Important appender functions
##### [exports.configure](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/lib/appenders/hipchat.js#L41)
Here is our `exports.configure` function.
```
function configure(_config) {

    if (_config.layout) {
        layout = layouts.layout(_config.layout.type, _config.layout);
    }

    hipchat = new HipChatClient(_config.api_key);

    return hipchatAppender(_config, layout);
}
```
Basically stated, this function configures the appender instance to set the custom layout for the appender if a layout was established in the json configuration.  It then instantiates the hipchat client for this loggers instance which will connect to hipchat with the specified api_key.  It will then return an instance of the hipchatAppender which informs log4js where to render the formatted text for hipchat.

##### [exports.appender](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/lib/appenders/hipchat.js#L20)

```
function hipchatAppender(_config, _layout) {

    layout = _layout || layouts.basicLayout;

    return function (loggingEvent) {

        var data = {
            room_id: _config.room_id,
            from: _config.from,
            message: layout(loggingEvent, _config.timezoneOffset),
            format: _config.format,
            color: colours[loggingEvent.level.toString()],
            notify: _config.notify
        };

        hipchat.api.rooms.message(data, function (err, res) {
            if (err) { throw err; }
        });
    };
}
```
This function establishes the layout/formatting function for the log4js event that occurs when the [LoggingEvent](https://github.com/chrispotter/log4js-node/blob/hipchat-connection/lib/logger.js#L18) is registered for the appenders function. If no layout is specified for the then the log4js basicLayout is used, which just generates a string with the format of [DATETIME][LOG_LEVEL]category - log text.

Then everytime a LoggingEvent is registered for this log4js instance it will trigger function which will build the `var data` from the registered configuraton for the appender, and the layout established, and pass the `data` to the `hipchat.api.message` function which logs the message to the hipchat client.  

#### Notables
`var colours` has been appended to this appender since the hipchat colors are more limited than the default log4js colors.  

`package.json` needs to include `hipchat-client` or when trying to log to hipchat an error will be thrown that will declare the hipchat module is not found.

###### Working Hipchat logger
![Normal](/2016/2/14/images/hipchat-log.png)


### To Be Continued
This was the first step to get logging to hipchat for [BlueAcornInc](https://github.com/BlueAcornInc), but I decided to try to give back to the greater node community by trying to create a pull request back into the original node4js repo.  

This required me to learn the javascript library `vows`.
