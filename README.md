minicron
=======

[![Build Status](http://img.shields.io/travis/jamesrwhite/minicron.svg)](http://travis-ci.org/jamesrwhite/minicron)
[![Code Climate](http://img.shields.io/codeclimate/github/jamesrwhite/minicron.svg)](https://codeclimate.com/github/jamesrwhite/minicron)
[![Dependency Status](http://img.shields.io/gemnasium/jamesrwhite/minicron.svg)](https://gemnasium.com/jamesrwhite/minicron)
[![Inline docs](http://inch-ci.org/github/jamesrwhite/minicron.png)](http://inch-ci.org/github/jamesrwhite/minicron)

minicron aims to complement ````cron```` by making it easier to manage and monitor cron jobs, it can largely be
thought of as two components that interact together, the CLI and the Hub. The CLI is what is installed on your
server(s) and executes your cron command and reports the status back to the Hub. The Hub is the central point
where data from one or many instances of the CLI is received and stored in a database. The Hub also provides
a web interface to the data and makes it easy to manage your cron jobs.

- [Roadmap](#roadmap)
- [Background](#background)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Security](#security)
- [Versioning](#versioning)
- [Contributing](#contributing)
- [Support](#support)
- [Credit](#credit)
- [License](#license)

Screenshots
------------

<img src="http://f.cl.ly/items/1k1h3n2A3Z3P3v2o0733/Image%202014-04-15%20at%2012.50.56%20am.png" height="175"/>
<img src="http://f.cl.ly/items/0c27341m2l1E230B0q1l/Image%202014-04-15%20at%2012.40.42%20am.png" height="175"/>
<img src="http://f.cl.ly/items/0Y2O0P0j012s3C3t3a3E/Image%202014-04-15%20at%2012.39.19%20am.png" height="175"/>
<img src="http://f.cl.ly/items/2R1f2m350W46423c220D/Image%202014-04-15%20at%2012.32.41%20am.png" height="175"/>

Roadmap
-------

After thinking a lot about the future of minicron and it's long overdue 1.0 release I've decided to focus it solely on 
monitoring cron jobs and doing that really well and leave the management side [to](https://www.chef.io)
[other](https://puppet.com/) [tools](https://www.ansible.com/). As such I'm removing the features related to SSH
based magement of jobs, it's possible that in future versions I may revist this but for now I want to focus solely
on monitoring. All development on this effort will be happening in the
[develop](https://github.com/jamesrwhite/minicron/tree/develop) branch and there will be no further 0.9.x releases.

Background
-----------

I initially developed minicron as part of my dissertation at university. The inspiration for developing minicron comes
largely from my experience and frustrations using cron, in particular my time spent working at [Miniclip](http://www.miniclip.com)
(which is where the name comes from, miniclip.. minicron, get it?) where the management and monitoring of cron jobs at times proved
to be tricky!

Features
---------

- Web UI
  - CRUD for cron jobs using ssh
  - GUI for cron schedule create/read/update
  - View realtime output/status as jobs run
  - Historical data for all job executions
- Alerts when jobs executions are missed or fail via:
  - Email
  - SMS ([using Twilio](https://www.twilio.com))
  - [PagerDuty](http://www.pagerduty.com) (SMS, Phone, Mobile Push Notifications and Email)
  - [Amazon Simple Notification Service](https://aws.amazon.com/sns)
  - [Slack](https://slack.com)

Lots more is planned for the future, see [open issues](https://github.com/jamesrwhite/minicron/issues?state=open) or if
you don't see the feature you want there add it!

Requirements
-------------

#### OS

*Should* run on OSX and any Linux/BSD based OS.

#### Database

**Default**

- SQLite

**Also Supported**

- MySQL
- PostgreSQL

#### Web Server / Reverse Proxy

#### Nginx
A simple [example config](config/nginx.conf) for nginx is provided.

#### Apache
If you're using apache as your reverse proxy you need to ensure you have the following modules installed:
- `libapache2-mod-proxy-html`
- `apache2-utils`

Run the following to enable them and then restart apache `a2enmod proxy proxy_html proxy_http xml2enc`.

A simple [example config](config/apache.conf) for apache is provided.

Installation
-------------

minicron is currently under heavy development and I make no gurantees about stability while it remains pre 1.0, as such
I would not recommend that you use this in production but I encourage you to give it a try in a non critical environment
and help me to improve it and work towards the first stable release.

minicron used to be available to install via [Ruby Gems](https://rubygems.org/gems/minicron), all future releases (>= 0.8)
will no longer be published there and made available as [ZIP file releases](https://github.com/jamesrwhite/minicron/releases)
instead.

#### Recommended

1. First check you meet the [requirements](#requirements)

2. Use the handy [install script](install.sh) or [grab the latest](https://github.com/jamesrwhite/minicron/releases/tag/v0.9.7) zip/tarball
   for your OS and install manually.
   ```
   bash -c "$(curl -sSL https://raw.githubusercontent.com/jamesrwhite/minicron/master/install.sh)"
   ```

3. Set your database configuration options in ````/etc/minicron.toml````, you can use the
   [minicron.toml](config/minicron.toml) as a guide on what options
   are configurable

  > **WARNING**
  >
  > the step below will drop any existing tables in the configured database and create new ones i.e:
  > ````sql
  > DROP TABLE IF EXISTS jobs; CREATE TABLE jobs ..
  > ````

4. You can then ````minicron db setup```` to create the database schema and run any pending migrations.

5. Done! See the usage section below for more details on how to use minicron now you have it installed

#### Docker

You can also run minicron in a docker container, see below for instructions how:

````bash
docker pull jamesrwhite/minicron:v0.9.7
minicron_container_id=$(docker run -d -p 127.0.0.1:9292:9292 -i -t jamesrwhite/minicron:v0.9.7)
docker exec $minicron_container_id minicron db setup
docker exec $minicron_container_id minicron server start
````

Usage
-----

#### Run a command

````bash
minicron run 'mysqldump db > backup.sql'
````

The global ````--verbose```` option can also be passed to the ````run```` argument like so

````bash
minicron run --verbose ls
````

You can also run a command in 'dry run' mode to test if it works without sending the output to the server.

````bash
minicron run --dry-run 'zip -r backup.zip website'
````

for further information see ````minicron help run````.

#### Get help

Running ````minicron```` with no arguments is an alias to running ````minicron help````, ````minicron -h```` or ````minicron --help````.
You can also use the help argument to get information on any command as shown above in the run a command section
or alternatively you can pass the ````-h```` or ````--help```` options like so ````minicron run -h````.

#### Server

To launch the server (aka the Hub) run

````bash
minicron server start
````

by default it will bind to port 9292 on the host 0.0.0.0 but this can be configured by the command line
arguments ````--host```` ````--port```` and ````--path```` or in the config file.

By default the server will run as a daemon with its process id stored in ````/tmp/minicron.pid````
you can also use the ````stop````, ````restart```` and ````status```` commands to control the server.

To run the server in debug mode, so you can see its output and any errors you can pass the ````--debug````
option.

See [nginx.conf](config/nginx.conf) for an example of
how to run minicron behind a reverse proxy.

#### Connecting to a host via SSH

To be able to perform CRUD operations on the crontab minicron needs to connect via SSH to the host.
When you set up a host minicron automatically creates a public/private key pair for you and stores it
in ````~/.ssh```` on the host the minicron server is being run on using the naming schema ````minicron_host_*HOST_ID*_rsa(.pub)````.
You will then to add the public key into the remote hosts authorized_keys file.

#### Version

Like many command line programs minicron will show its version number when the global options ````-v````
or ````--version```` are passed to the CLI.

#### Configuration

Some configuration options can be passed in manually but the recommend way to configure minicron is through the use
of a config file. You can specify the path to the file using the ````--config```` global option. The file is expected
to be in the [toml](https://github.com/mojombo/toml) format. The default options are specified in the
[minicron.toml](config/minicron.toml)
file and minicron will parse a config located in ````/etc/minicron.toml```` if it exists. Options specified via
the command line will take precedence over those taken from a config file.

Security
---------

As mentioned previously minicron is still under development and as such is missing some essential features as far as
security is concerned. For example authentication still needs to be added.

  > **It is not recommended that you allow your minicron host to be accessible via the public internet!**

Obviously without authentication anyone who knew the address of your minicron host would be able to set up
a potentially malicious job on one of your servers! Future versions may be secure enough to expose publicly but personally
I still would not recommend it, minicron is designed to be an internal tool and should be behind a firewall that only allows
connections from an internal network and/or a VPN.

Versioning
-----------

All stable releases will follow the [semantic versioning](http://semver.org/) guidelines. Until 1.0 hits I will try
to document any breaking changes in the release descriptions but you should proceed with caution before relying
on anything etc etc

Releases will be numbered with the following format:

`<major>.<minor>.<patch>`

Based on the following guidelines:

* A new *major* release indicates a large change where backwards compatibility is broken.
* A new *minor* release indicates a normal change that maintains backwards compatibility.
* A new *patch* release indicates a bugfix or small change which does not affect compatibility.

Contributing
------------

Feedback and pull requests are welcome, please see [CONTRIBUTING.md](CONTRIBUTING.md)
for more info.

Areas that I would love some help with:

- Any of the unassigned [issues here](https://github.com/jamesrwhite/minicron/issues?state=open).
- General testing of the system, let me know what you think and create issues for any bugs you find!
- Tests!!
- Validation and error handling improvements
- Documentation improvements.
- Look for '[TODO:](https://github.com/jamesrwhite/minicron/search?q=TODO%3A)' notices littered around the code,
  I'm trying to convert them all to issues but there are a lot..
- Code refactoring, I had a deadline to meet for the initial versions so some parts are a tad rushed
- UI improvements

Support
--------

Where possible I will try and provide support for minicron but I offer no gurantees etc.

Feel free to open an issue and I'll do my best to help.

Credit
-------

minicron makes use of a *lot* of awesome open source projects that have saved me a lot of time in its development.
I started out trying to list all of them but it was taking way too much time so check out the dependencies in
[minicron.gemspec](minicron.gemspec) and
[app.rb](lib/minicron/hub/app.rb).

License
--------

minicron is licensed under the GPL v3, [see here for the full license](LICENSE)
