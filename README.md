# Swoole stats by Zabbix Agent

Template for monitoring swoole server stats (and some extras) with Zabbix, expects output of [Swoole\Server::stats](https://www.php.net/manual/en/swoole-server.stats.php) in JSON format.

## How to make this work:

Import the template on the **Data collection > Templates** page.

Install Zabbix Agent on the host where the swoole server is running, or from which the swoole stats will be available (you probably don't want them public).

Add that host to **Data collection > Hosts**, add the imported template to that host.

In the **Host > Macros** set:

 - `{$SWOOLE.PROCESS_CDMLINE}` - full cmdline of the swoole server process as shown in `ps`. For example use the output of `ps -ax -o args | grep php`, mine is `php8.3 artisan octane:start`. Hope you know how you started the swoole server. This is used to collect CPU and memory usage with the agent itself, not used for gathering swoole stats themselves. May be empty or unset, the relevant items just won't be collected.
 - `{$SWOOLE.PROCESS_USER}` - an additional filter for the case you have more than one swoole server running. May be empty or unset.Probably should not be set with empty `{$SWOOLE.PROCESS_CDMLINE}`, Zabbix Agent may collect all of that user's usage instead of the single swoole server process.
 - `{$SWOOLE.SERVER_STATUS.HOST}` - the domain/IP part of the swoole stats URL, defaults to `localhost`.
 - `{$SWOOLE.SERVER_STATUS.PATH}` - the URI part of the swoole stats URL, defaulst to `/swoole-stats`.
 - `{$SWOOLE.SERVER_STATUS.PORT}` - a port the swoole server listens on, defaults to `8000`.

The last three items defaults result in `localhost:8000/swoole-stats` the `http://` part is implied by the Zabbix item type used.

The other important part is to make your swoole server output the stats, it's not the default behaviour. The [Swoole\Server::stats](https://www.php.net/manual/en/swoole-server.stats.php) function outputs the stats array which then must be served as a web page in json format for Zabbix to be able to parse it. I have this done by a php developer (which I am not) in Laravel Octane. In the end you should be able to `curl http://localhost:8000/swoole-stats` or your actual URL from the host where the Agent is installed and the output should be a JSON object and nothing else. 
