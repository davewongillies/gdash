What?
=====

A simple Graphite dashboard built using Twitter's Bootstrap.

Adding new dashboards is very easy and individual graphs are
described using a small DSL.

See the _sample_ directory for a sample dashboard configuration.

![Sample dashboard](https://github.com/ripienaar/gdash/raw/master/sample/email.png)

Install?
-------

This dashboard is a Sinatra application, I suggest deploying it
in Passenger or other Sinatra application server.

You can also build an executable WAR to run gdash with JRuby. (experimental)


## JRuby? (experimental)

To run gdash with JRuby, e.g. if you have to run in on a windows machine, you can create a (executable) WAR archive.
Currently you still need Ruby for creating the archive.	
This is still experimental!

###Known Issues

- I had to remove the dependency to redcarpet (because of native C dependencies, see https://github.com/jruby/jruby/wiki/Troubleshooting), so rendering the "docs" page will fail. 
- I experienced crashes of the packaged jetty during my tests

###Todo:
- Replace redcarpet with a different renderer or make it work with JRuby/warble
- Do proper testing on different platforms
- Speed up startup time, performance in general

###How to create a (executable) WAR:

```
git clone --branch=jruby https://github.com/ripienaar/gdash.git	
cd gdash  	
gem install bundler  
bundle install   
```

Configure your environment (see below, gdash.yaml must exist!). Make sure no gdash.war redisdes in your current directory. 	
`rake war  `
or  
`warble executable war  `
if you want to include a webserver (default: jetty, edit /config/warbl.rb to switch to winstone or jenkins webserver).  

Run (example for jetty):	
`	java -Djetty.port=8080 -Dconfig=<absolutepath>/gdash.yaml -jar gdash.war`



Config?
-------

A sample _gdash.yaml-sample_ is included, you should rename it to
_gdash.yaml_ and adjust the url to your Graphite etc in there.

The SinatraApp class take two required arguments:

    * Where graphite is installed
    * The directory that has your _dashboards_ directory full of templates

and additional options:

    * The title to show at the top of your Graphite
    * A prefix to prepend to all URLs in the dashboard
    * How many columns of graphs to create, 2 by default.
    * How often dashboard page is refreshed, 60 sec by default.
    * The width of the graphs, 500 by default
    * The height of the graphs, 250 by default
    * Where your whisper files are stored - future use
    * Optional interval quick filters

Creating Dashboards?
--------------------

You can have multiple top level categories of dashboard.  Just create directories
in the _templatedir_ for each top level category.

In each top level category create a sub directory with a short name for each new dashboard.

You need a file called _dash.yaml_ for each dashboard, here is a sample:

    :name: Email Metrics
    :description: Hourly metrics for the email system

Then create descriptions in files like _cpu.graph_ in the same directory, here
is a sample:

    title       "Combined CPU Usage"
    vtitle      "percent"
    area        :stacked
    description "The combined CPU usage for all Exim Anti Spam servers"

    field :iowait, :scale => 0.001,
                   :color => "red",
                   :alias => "IO Wait",
                   :data  => "sumSeries(derivative(mw*munin.cpu.iowait))"

    field :system, :scale => 0.001,
                   :color => "orange",
                   :alias => "System",
                   :data  => "sumSeries(derivative(mw*.munin.cpu.system))"

    field :user, :scale => 0.001,
                 :color => "yellow",
                 :alias => "User",
                 :data  => "sumSeries(derivative(mw*.munin.cpu.user))"

The dashboard will use the _description_ field to show popup information bubbles
when someone hovers over a graph with their mouse for 2 seconds.

The graphs are described using a DSL that has its own project and documented
over at https://github.com/ripienaar/graphite-graph-dsl/wiki

At the moment we do not support the _Related Items_ feature of the DSL.

Template Directory Layout?
--------------------------

The directory layout is such that you can have many groupins of dashboards each with
many dashboards underneath it, an example layout of your templates dir would be:

        graph_templates
        `-- virtualization
            |-- dom0
            |   |-- dash.yaml
            |   |-- iowait.graph
            |   |-- load.graph
            |   |-- system.graph
            |   |-- threads.graph
            |   `-- user.graph
            `-- kvm1
                |-- dash.yaml
                |-- disk_read.graph
                |-- disk_write.graph
                |-- ssd_read.graph
                `-- ssd_write.graph

Here we have a group of dashboards called 'virtualization' with 2 dashboards inside it
each with numerous graphs.

You can create as many groups as you want each with many dashboards inside.

Custom Time Intervals?
--------------------

You can reuse your dashboards and adjust the time interval by using the following url
structure:

    http://gdash.example.com/dashboard/email/time/-8d/-7d

or

    http://gdash.example.com/dashboard/email/?from=-8d&until=-7d
    http://gdash.example.com/dashboard/email/full/2/600/300?from=-8d&until=-7d

This will display the _email_ dashboard with a time interval same day last week.
If you hit */dashboard/email/time/* it will default to the past hour (*-1hour*)
See http://graphite.readthedocs.org/en/1.0/url-api.html#from-until for more info
acceptable *from* and *until* values.

Quick interval filters shown in interface are configurable in _gdash.yaml_ options sections. Eg:

	:options:
           :interval_filters:
             - :label: Last Hour
               :from: -1h
               :to: now
             - :label: Last Day
               :from: -1day
             - :label: Current Week
               :from: monday
               :to: now

Quick filter is not shown when *interval_filters* section is missing in configuration file.

Time Intervals Display?
-----------------------

If you configure time intervals in the config file you can click on any graph in
the main dashboard view and get a view with different time intervals of the same
graph

	:options:
	  :intervals:
	    - [ "-1hour", "1 hour" ]
	    - [ "-2hour", "2 hour" ]
	    - [ "-1day", "1 day" ]
	    - [ "-1month", "1 month" ]
	    - [ "-1year", "1 year" ]

With this in place in the _config.yaml_ clicking on a graph will show the 5 intervals
defined above of that graph

Full Screen Displays?
---------------------

You can reuse your dashboards for big displays against a wall in your NOC or office
by using the following url structure:

    http://gdash.example.com/dashboard/email/full/4/600/300
    http://gdash.example.com/dashboard/email/full/4?width=600&height=300

This will display the _email_ dashboard in _4_ columns each graph with a width of
_600_ and a height of _300_

The screen will refresh every minute

Define several graphite backends?
--------------------------------

You can overwrite the default graphite setting from gdash.yaml setting :graphite: in the the dash.yaml:

    :graphite: http://mygraphitehost:80

Additional properties in graphs?
--------------------------------

You can specify additional properties in the dash.yaml for each dashboard:

    :graph_properties:
        :environment: dev
        :servers: [ "server1.domain.com", "server2.domain.com" ]
        :javaid: 1234
    
that can be accessed from the .graph like:
    
    servers = @properties[:servers]
    environment = @properties[:environment]
    
    field :iowait, 
        :alias => "IO Wait #{server}",
        :data  => "servers.#{environment}.#{servers.join(',')}.cpu*.cpu-{system,wait}.value"

Include graphs from other dashboard?
------------------------------------

You can include the graphs from other dashboard with the include 
property in dash.yaml:

    :include_graphs: 
    - "templates/os.basic" 
    - "templates/os.nfs" 

Load dashboard properties from a external YAML file? 
----------------------------------------------------

If you got a set of common properties that you want to reuse in the 
dashboard, you can load a external yaml file from in dash.yaml. 
The path is relative to the _templatedir_ and it does not support 
recursive includes.

Examples are a list of server colors, timezones, etc. In _dash.yaml_:

    :include_properties: 
     - "common.yml" 
     - "black-theme.yml" 

Example _common.yml_:

    :graph_properties: 
     :timezone:         Europe/London
     :hide_legend:      false

Example _black-theme.yml_:

    :graph_properties: 
     :background_color: white
     :foreground_color: black
     :vertical_mark_color: "#330000"

A external properties files can be also loaded from the url:

  http://graphite.example.net:3000/category_name/dash_name/?include_properties=white-theme.yml

Special properties when printing?
---------------------------------

When printing these properties will be overrided:  

    :graph_properties: 
     :background_color: white
     :foreground_color: black

You can create an optional YAML file _templatedir/print.yml_ that will be loaded when printing.
This way you can override additional properties or use custom colors for printing.

Placeholder Parameters
----------------------

Provide variables in the URL:

  http://graphite.example.net:3000/category_name/dash_name/?p[node]=node.example.net

And use them in the graphs

    field :iowait, 
        :data  => "servers.%{node}.cpu*.cpu-wait.value"

It will also override any graph properties as in the :graph_properties: in dash.yaml, so
the value can be accessed using the @properties hash:

    node = @properties[:node]

Also can be used to override graph properties like the timezone:

  http://graphite.example.net:3000/category_name/dash_name/?p[timezone]=CET

Contact?
--------

R.I.Pienaar / rip@devco.net / http://www.devco.net/ / @ripienaar
