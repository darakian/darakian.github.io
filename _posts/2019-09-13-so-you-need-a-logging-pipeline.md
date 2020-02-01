---
layout: post
title:  "So you need a logging pipeline"
date:   2019-09-13 18:00:00 -0700
---
# You have chosen or have been chosen to implement a logging pipeline
So, some event has occurred within your organisation and now you need a logging pipeline. What do you do? Where do you begin? Most people come to holistic logging from some shock event like a hack event, a catastrophic failure, or some external requirement. Whether you're reading this as a result of something tragic or simply trying to prepare I'll share a story and some knowledge gained during my experience. Generally speaking this post will revolve around Mozilla's fantastic [MozDef](https://github.com/mozilla/MozDef) product which is a set of docker containers that work together to provide a logging sink, alerting logic and manual inspection interface.

# Pipeline requirements
In my case the primary task was to enable real time alerting on all logs coming out of our infrastructure. At the time we had a central logging sink provided by a third party which was in essence a dumping ground for log messages. We collected everything but then just dumped it on a server for some basic grep interactive like parsing later on. If you've come across the term [SIEM](https://en.wikipedia.org/wiki/Security_information_and_event_management), that's what I needed.

# Application Logging vs System Logging
There are broadly two types of logging to consider when setting everything up: Application logging and System logging. Application logs are those logs which are generated within your business logic and which can be made to avoid the general system logger. If you can setup application logging then do so. The logs you will get out of application logging have a very good signal to noise ratio. System logging on the other hand refers to everything running on the system that would be picked up by the host system and is high noise as a result. It's fairly common for business logic applications to log to the system, but it's worth exploring your particular tech stack to see if a new logger can be added to the business logic itself to reduce noise.

# Logs are inherently unreliable
You will lose logs. Sorry, but it will happen. Routing tables will be incorrectly configured, firewalls will be out of your control, deployed images or containers will be missing files or programs, logging code will have bugs and will crash, etc... Don't worry about missing a few logs here and there. Take it as a given that logs will go missing and you should account for that. For what it's worth missing logs can be some of the most informative of all.

# Transport options
Ok, so how should you emit your logs? Simple answer: http. Well https, but unless you have good reason to just emit json over http. Yes there's overhead, but every tool you will come across supports the combination of json over http and you get a protocol that firewalls usually don't interfere with. Additionally http gives you arbitrary headers with which you can customise routing and simple authentication of logs. For example, you might find yourself in the situation where you need to accept logging input from networks outside of your own. You may also find yourself with no infrastructure in place to provision private keys and a deep desire to avoid white listing. If that's the case make a header value along the lines of `X-My-Log-Input-Key` with a value along the lines of `ad1071715c1936bb308ea46a298c1b82132bba604eae633a280fd326f442b0e0` (created with `head /dev/random | shasum -a 512256`) and reject all http traffic for which that header is not correct. Obviously keep that header value secret and rotate as often as possible.

# (R)syslog over http
While it's nice to avoid if possible you may find yourself in the same unenviable situation as I did where all system logging needed to be captured and analysed. In the event this is the case, I'm just going to recommend rsyslog with the omhttp module. It may be a bit annoying to get the modern rsyslog integrated into your infrastructure, but it works on all major linux distros and lets you customise output. There are more and probably better options here, but rsyslog will get the job done. A simplified version of my rsyslog config can be found below.

# Application logging over http
This will be highly language and framework dependent, but as long as you have a logging subsystem you have the ability to make ship logs out of your application. Python in particular has a very nice logging framework where custom log handlers can be made and registered to provide custom output options. [Python logging handlers](https://docs.python.org/3.7/library/logging.handlers.html). A simple example of a mozdef compatible logging handler can be found below.

# Mozilla's Marvellous Mozdef
Mozdef is essentially a [ELK stack](https://www.elastic.co/what-is/elk-stack) with an analysis framework added on. This "MELK" stack provides all the normal elastic co goodness with a framework to schedule and run arbitrary queries on your log data. That and it's dead simple to get going. Assuming you have docker and docker-compose installed, a basic setup can be started with
```
git clone https://github.com/mozilla/MozDef.git
cd MozDef
make run
```
Why it's controlled via makefile I'm not quite sure, but I don't really care either. With that three line invocation you'll be running a set of docker containers which accept json over http on `localhost:8080/events`, an alerting interface available at `localhost:80` and a kibana search interface on `localhost:9090`. If all you need is a basic log sink then you're done. If you need to override environmental variables you can issue `make run-env-mozdef -e ENV=my.env` rather than `make run`. In the background mozdef will start up an elastic search database for itself, but by passing in an env file you can configure your mozdef instance to use an external elastic search DB. Similarly you can override the use of the built in kibana instance if you like. See the mozdef docs for more on what specifically can be overridden.

# Alerting
While having an elk stack in a box is nice, the main draw to mozdef in particular is the alerting framework. In essence you write a python script to query your logs based on whatever metrics you care about and create alert objects based on the results. The alert objects will be passed through to the mozdef web interface and will be available for issue triage and tracking. There's a simple on-boarding guide for writing alerts available here: [Example first alert](https://mozdef.readthedocs.io/en/latest/alert_development_guide.html#example-first-alert).

# Wrap up
If you found yourself looking for an introduction to logging then I hope you've found this useful. Below are the rsyslog and python logging configs that I've used.
```
Thanks for reading
```

### Rsyslog output for mozdef
```
# Define json template.
$template mozdef_json,"{\"category\":\"syslog\",\"tags\":\"%SYSLOGTAG%\",\"processname\":\"%APP-NAME%\",\"processid\":\"%PROCID%\",\"PRI\":\"%PRI%\",\"PRI-Text\":\"%PRI-TEXT%\",\"severity\":\"%SYSLOGSEVERITY%\",\"hostname\":\"%HOSTNAME%\",\"summary\":\"%msg:::json%\",\"message id\":\"%MSGID%\",\"structured data\":\"%STRUCTURED-DATA%\"}"

module(load="omhttp")
action(
    type="omhttp"
    server="mymozdef.foo.bar.com"
    serverport="8080"
    restpath="events"
    dynRestPath="off"
    compress="off"
    usehttps="on"
    template="mozdef_json"
    httpheaderkey="X-My-Log-Input-Key"
    httpheadervalue="ad1071715c1936bb308ea46a298c1b82132bba604eae633a280fd326f442b0e0"
    errorfile="/var/log/rsyslog-mozdef-http-errors")
```
Full documentation for the omhttp module can be found here [Rsyslog omhttp](https://www.rsyslog.com/doc/v8-stable/configuration/modules/omhttp.html)

### Simple mozdef logging handler for python

```
import logging
import requests
import json
import socket

class MozDefHandler(logging.Handler):
    def __init__(self, broker=None, logging_key=None):
        super(MozDefHandler, self).__init__()
        self.broker = broker
        self.logging_key = logging_key
        self.thread_pool_executor = ThreadPoolExecutor(max_workers=1)

    def emit(self, record):
        log_entry = self.format(record)
        self.thread_pool_executor.submit(
            self.ship_log,
            log_dest=self.broker,
            log_entry=log_entry,
            record=record,
            logging_key=self.logging_key
        )

    def ship_log(self, log_dest=None, log_entry=None, record=None, logging_key=None):
        try:
            requests.post(log_dest,
                          headers={
                            'Content-type': 'application/json',
                            'Connection': None,
                            'X-Log-Key': logging_key
                          },
                          data=log_entry,
                          timeout=0.5
                          )
        except Exception:
            self.handleError(record)


class MozDefFormatter(logging.Formatter):
    def __init__(self, level=logging.INFO, category='web'):
        super(MozDefFormatter, self).__init__()
        self.category = category

    def format(self, record):
        # https://docs.python.org/3/library/logging.html#logrecord-attributes
        # https://mozdef.readthedocs.io/en/latest/usage.html#mandatory-fields
        dict_record = {}
        dict_record['category'] = self.category
        dict_record['hostname'] = socket.gethostname()
        dict_record['tags'] = []
        dict_record['module'] = record.module
        dict_record['source'] = record.pathname
        dict_record['severity'] = record.levelname
        dict_record['processid'] = record.process
        dict_record['processname'] = record.module
        dict_record['timestamp'] = record.created
        dict_record['details'] = {
                            'relativeCreatedTimestamp': record.relativeCreated,
                            'loglevel': record.levelno,
                            'linenumber': record.lineno
                            }
        dict_record['summary'] = record.getMessage()
        return json.dumps(dict_record)
```
