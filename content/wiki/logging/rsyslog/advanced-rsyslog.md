---
{"publish":true,"title":"Advanced RSyslog","created":"2025-08-29","modified":"2025-08-30T11:00:20.858-04:00","published":"2025-08-29","tags":["logging","rsyslog"],"cssclasses":""}
---

# Purpose and Design

This configuration example utilizes the power of Rsyslog v7.x's RainerScript as well as Logstash and ElasticSearch for later analysis.

# Configuration

**rsyslog.conf:**

~~~~~
# rsyslog v7 configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### MODULES ####

module(load="imuxsock") # provides support for local system logging (e.g. via logger command)
module(load="imklog")   # provides kernel logging support (previously done by rklogd)
#module(load"immark")  # provides --MARK-- message capability

module(load="mmnormalize")
module(load="omelasticsearch")

# Provides UDP syslog reception
# for parameters see http://www.rsyslog.com/doc/imudp.html
#module(load="imudp") # needs to be done just once
#input(type="imudp" port="514")
module(load="imudp")

# Provides TCP syslog reception
# for parameters see http://www.rsyslog.com/doc/imtcp.html
#module(load="imtcp") # needs to be done just once
#input(type="imtcp" port="514")
module(load="imtcp" MaxSessions="500")
module(load="imrelp" RuleSet="remote")


#### GLOBAL DIRECTIVES ####

# Templates
template(name="RemoteHost" type="string" string="/srv/log/%HOSTNAME%/%$YEAR%/%$MONTH%/syslog-%$DAY%.log")

$IncludeConfig /etc/rsyslog.d/*.template

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Include all config files in /etc/rsyslog.d/
$IncludeConfig /etc/rsyslog.d/*.conf


#### RULES ####

ruleset(name="local") {
    # Log all kernel messages to the console.
    # Logging much else clutters up the screen.
    kern.*                                                  /var/log/kern.log

    # Log anything (except mail) of level info or higher.
    # Don't log private authentication messages!
    *.info;mail.none;authpriv.none;cron.none                /var/log/messages

    # The authpriv file has restricted access.
    authpriv.*                                              /var/log/secure

    # Log all the mail messages in one place.
    mail.*                                                  -/var/log/maillog


    # Log cron stuff
    cron.*                                                  /var/log/cron

    # Everybody gets emergency messages
    *.emerg                                                 :omusrmsg:*

    # Save news errors of level crit and higher in a special file.
    uucp,news.crit                                          /var/log/spooler

    # Save boot messages also to boot.log
    local7.*                                                /var/log/boot.log

    *.* action(type="omelasticsearch"
               server="localhost"
               serverport="9200"
               template="logstash"
               searchIndex="logstash-index"
               dynSearchIndex="on"
               searchType="syslog"
               bulkmode="on"
               queue.type="linkedlist"
               queue.size="5000"
               queue.dequeuebatchsize="300"
               action.resumeretrycount="-1")
}

$DefaultRuleset local


ruleset(name="remote") {
    $IncludeConfig /etc/rsyslog.d/*.remote

    action(type="omfile" DynaFile="RemoteHost")
}

input(type="imudp" port="514" ruleset="remote")
input(type="imtcp" port="514" ruleset="remote")


$InputRELPServerBindRuleset remote
input(type="imrelp" port="20514")
~~~~~

**rsyslog.d/logstash.template:**

~~~~~
template(name="logstash-index"
         type="list") {
           constant(value="logstash-")
           property(name="timereported" dateFormat="rfc3339" position.from="1" position.to="4")
           constant(value=".")
           property(name="timereported" dateFormat="rfc3339" position.from="6" position.to="7")
           constant(value=".")
           property(name="timereported" dateFormat="rfc3339" position.from="9" position.to="10")
         }

template(name="logstash"
         type="list"
         option.json="on") {
           constant(value="{")
             constant(value="\"@timestamp\":\"")     property(name="timereported" dateFormat="rfc3339")
             constant(value="\",\"message\":\"")     property(name="msg")
             constant(value="\",\"host\":\"")        property(name="hostname")
             constant(value="\",\"severity\":\"")    property(name="syslogseverity-text")
             constant(value="\",\"facility\":\"")    property(name="syslogfacility-text")
             constant(value="\",\"syslogtag\":\"")   property(name="syslogtag")
           constant(value="\"}")
         }

template(name="logstash-accesslog"
         type="list"
         option.json="on") {
           constant(value="{")
             constant(value="\"@timestamp\":\"")           property(name="timereported" dateFormat="rfc3339")
             constant(value="\",\"message\":\"")           property(name="msg" position.from="2" spifno1stsp="off")
             #constant(value="\",\"message\":\"")           property(name="msg" position.from="2" spifno1stsp="off" format="json")
             constant(value="\",\"host\":\"")              property(name="fromhost-ip")
             constant(value="\",\"@source_host\":\"")      property(name="hostname")
             constant(value="\",\"vhost\":\"")             property(name="$!vhost")
             constant(value="\",\"@fields\": {")
               constant(value="\"bytes\":\"")              property(name="$!bytesend")
               constant(value="\",\"clientip\":\"")        property(name="$!ip")
               constant(value="\",\"method\":\"")          property(name="$!method")
               constant(value="\",\"request\":\"")         property(name="$!url")
               constant(value="\",\"referrer\":\"")        property(name="$!referer")
               constant(value="\",\"useragent\":\"")       property(name="$!useragent")
               constant(value="\",\"status\":\"")          property(name="$!status")
             constant(value="\"}")
           constant(value="}")
         }

template(name="logstash-errorlog"
         type="list"
         option.json="on") {
           constant(value="{")
             constant(value="\"@timestamp\":\"")           property(name="timereported" dateFormat="rfc3339")
             constant(value="\",\"message\":\"")           property(name="msg" position.from="2" spifno1stsp="off")
             constant(value="\",\"host\":\"")              property(name="fromhost-ip")
             constant(value="\",\"@source_host\":\"")      property(name="hostname")
             constant(value="\",\"severity\":\"")          property(name="syslogseverity-text")
           constant(value="\"}")
         }
~~~~~

**rsyslog.d/webservers.template:**

~~~~~
template(name="httpd-access" type="list") {
    property(name="msg" position.from="2" spifno1stsp="off")
    #property(name="msg" droplastlf="on")
    constant(value="\n")
}
template(name="httpd-error" type="list") {
    constant(value="&lt;")
    property(name="syslogpriority-text")
    constant(value="&gt; ")
    property(name="timestamp" dateFormat="rfc3339")
    constant(value=" ")
    property(name="hostname")
    constant(value=" ")
    property(name="syslogtag" position.from="1" position.to="32")
    property(name="msg" spifno1stsp="off")
    property(name="msg" droplastlf="on")
    constant(value="\n")
}

template(name="WebErrFiles" type="string" string="/srv/log/WEB/%$YEAR%/%$MONTH%/httpd_error-%$DAY%.log")
template(name="PhpErrFiles" type="string" string="/srv/log/WEB/%$YEAR%/%$MONTH%/php_error-%$DAY%.log")
template(name="WebFiles" type="string" string="/srv/log/WEB/%$YEAR%/%$MONTH%/access_%$!vhost%-%$DAY%.log")
~~~~~

**rsyslog.d/apacheaccess.rule:**

~~~~~
rule=: %vhost:word% %ip:word% %rlogname:word% %ruser:word% [%date:word% %heure:word% %tz:char-to:]%] "%method:word% %url:word% %pver:char-to:"%" %status:word% %bytesend:word% %referer:word% %useragent:quoted-string% %ssl:word% %sslport:word% %sslproto:word%
~~~~~

**rsyslog.d/10-webservers.remote:**

~~~~~
if $programname == 'httpd' or $programname == 'php' then {
    #if $syslogseverity-text == 'error' then {
    if $programname == 'php' then {
        action(type="omfile" DynaFile="PhpErrFiles" template="httpd-error")
        action(type="omelasticsearch"
               server="localhost"
               serverport="9200"
               template="logstash-accesslog"
               searchIndex="logstash-index"
               dynSearchIndex="on"
               searchType="php-errorlog"
               bulkmode="on"
               queue.type="linkedlist"
               queue.size="5000"
               queue.dequeuebatchsize="300"
               action.resumeretrycount="-1")
    } else if $syslogfacility-text == 'local1' then {
        action(type="omfile" DynaFile="WebErrFiles" template="httpd-error")
        action(type="omelasticsearch"
               server="localhost"
               serverport="9200"
               template="logstash-errorlog"
               searchIndex="logstash-index"
               dynSearchIndex="on"
               searchType="httpd-errorlog"
               bulkmode="on"
               queue.type="linkedlist"
               queue.size="5000"
               queue.dequeuebatchsize="300"
               action.resumeretrycount="-1")
        #action(type="omfwd" Target="172.17.51.4" Port="514" Protocol="tcp")
    } else {
        action(type="mmnormalize" userawmsg="off" rulebase="/etc/rsyslog.d/apacheaccess.rule")
        action(type="omfile" DynaFile="WebFiles" template="httpd-access")
        action(type="omelasticsearch"
               server="localhost"
               serverport="9200"
               template="logstash-accesslog"
               searchIndex="logstash-index"
               dynSearchIndex="on"
               searchType="accesslog"
               bulkmode="on"
               queue.type="linkedlist"
               queue.size="5000"
               queue.dequeuebatchsize="300"
               action.resumeretrycount="-1")
        #action(type="omfwd" Target="172.17.51.4" Port="514" Protocol="tcp")
    }
    stop
}
~~~~~

**rsyslog.d/90-logstash.remote:**

~~~~~
*.* action(type="omelasticsearch"
           server="localhost"
           serverport="9200"
           template="logstash"
           searchIndex="logstash-index"
           dynSearchIndex="on"
           searchType="syslog"
           bulkmode="on"
           queue.type="linkedlist"
           queue.size="5000"
           queue.dequeuebatchsize="300"
           action.resumeretrycount="-1")
~~~~~
