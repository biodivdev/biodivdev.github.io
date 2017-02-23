
One main improvement of latest update is the review of the whole configuration of the RRAPP.

Since there has been some interest on running it by different parties (this is the intention after all), it was clear that the current way to run it was overly complicated.

This post describe a initial approach to simplify this work.

## Published data

The main requirement to be able to run RRAPP is actually a data availability issue. You MUST have:

- A taxonomic checklist published on IPT
- Occurrences with coordinates published on IPT

## Composition of apps

One thing that is important to consider is that RRAPP is not a single tool, but actually a composition of three parts:

- The data bot
- The data analyzer
- The web interface

All along with a ElasticSearch database and a Proxy to better serve the content.

As such there is two approaches: Run all as whole (using docker-compose) or running individual parts.

### The ElasticSearch database

RRAPP uses the latest ElasticSearch 5.0 release. This is the single storage/database used. 

A common problem when running newest ES is "max virtual memory areas vm.max\_map\_count [65530] likely too low", that is solved by running "sudo sysctl -w vm.max\_map\_count=262145". But this change is transient, and will be overriden at reboot. To make it permanent, as root: "echo vm.max\_map\_count=262145 > /etc/sysctl.d/50-es.conf".

### The Proxy

Not mandatory if you are running each app on it's own, but useful anyway, is the proxy. Here the choice is Caddyserver, a new proxy server capable of automaticaly serving HTTPS sites and easy to configure.

In the compose repository, it is configured at config/Caddyfile, which tells to which address the proxy will bind and serve, defaulting to http://localhost:8080.

It can be configured to simply bind to a port (eg.: ":8080") or to a domain (e.g "https://br.biodiversity.cloud"), if you wish to enable HTTPS you must also provide the directive "tls email@domain.com" in any other line.

### Configuration files

#### dwc-bot.list

This file consist of a list of IPT URLs for the bot to crawl and index, it is simply a list with a single url in each line that points to a IPT intallation.

#### config.ini

The three main app use this file to run, and the bot and the analyzer(idx) will reload it on rerun.

It configures:

- ELASTICSEARCH: the address of the elasticsearch server
- INDEX: The index (database) to use on elasticsearch. Will be created if needed.
- SOURCE: The name of the crawled resource from which to load the taxonomic checklist
- LOOP: Used by the bot and idx to run only once or keep running.

When running the apps individually, all theses parameters can also be overriden by environment variables of java properties on execution, and so is the location of the file on env var CONFIG.

## Conclusion

This is a work in progress, but as always if you have problems or suggestions you can interact in Github with the issues pages of each project or direct with email with diogo@diogok.net.

