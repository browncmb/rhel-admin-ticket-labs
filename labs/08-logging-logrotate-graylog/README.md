# Centralized Logging and HTTPD Log Rotation

## Objective

Configure centralized Linux log forwarding, validate log ingestion in Graylog, troubleshoot missing log input behavior, and implement HTTPD log rotation for predictable operational log retention.

## Environment

* Red Hat-based Linux virtual machines
* rsyslog client configuration
* Graylog centralized logging platform
* Apache HTTP Server
* logrotate
* systemd timers
* firewalld
* Sanitized hostnames, IP addresses, URLs, usernames, queries, and log contents

## Work Performed

* Configured Linux systems to forward syslog events to a centralized Graylog server.
* Opened the required syslog listener port where appropriate.
* Restarted rsyslog after updating forwarding configuration.
* Generated test log events from Linux clients using `logger`.
* Investigated missing log ingestion and identified that the Graylog Syslog UDP input was not running.
* Started the required Graylog input and re-tested message forwarding.
* Verified centralized log ingestion from managed Linux systems.
* Queried Graylog for a service installation event.
* Configured HTTPD log rotation for daily rotation and 14 retained logs.
* Validated the logrotate policy with a forced rotation test.
* Confirmed logrotate scheduling through the systemd timer.

## Commands and Configuration

Example rsyslog forwarding configuration:

```bash
sudo vi /etc/rsyslog.d/10-graylog-forwarding.conf
```

```rsyslog
*.* @<graylog_server_ip>:5140;RSYSLOG_SyslogProtocol23Format
```

Restart rsyslog:

```bash
sudo systemctl restart rsyslog
sudo systemctl status rsyslog --no-pager
```

Allow the Graylog Syslog UDP listener port where required:

```bash
sudo firewall-cmd --permanent --add-port=5140/udp
sudo firewall-cmd --reload
```

Generate a test syslog message:

```bash
logger "graylog forwarding validation from <managed_node>"
```

Example sanitized Graylog search query:

```text
source:<managed_node> AND message:"graylog forwarding validation"
```

Example service installation search:

```text
source:<managed_node> AND message:<service_name> AND message:install
```

Example HTTPD logrotate policy:

```bash
sudo vi /etc/logrotate.d/httpd
```

```conf
/var/log/httpd/*log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    sharedscripts
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```

Force logrotate for validation:

```bash
sudo logrotate -vf /etc/logrotate.d/httpd
```

Check logrotate timer status:

```bash
systemctl status logrotate.timer --no-pager
systemctl list-timers | grep logrotate
```

## Validation

Confirm rsyslog is active after configuration:

```bash
systemctl status rsyslog --no-pager
```

Expected sanitized result:

```text
Active: active (running)
```

Confirm the Graylog listener port is allowed where applicable:

```bash
sudo firewall-cmd --list-ports
```

Expected sanitized result:

```text
5140/udp
```

Confirm test messages are searchable in Graylog:

```text
source:<managed_node> AND message:"graylog forwarding validation"
```

Expected sanitized result:

```text
<timestamp> <managed_node> graylog forwarding validation from <managed_node>
```

Confirm HTTPD logs rotate successfully:

```bash
ls -lh /var/log/httpd/
```

Expected sanitized result:

```text
access_log
access_log.1
error_log
error_log.1
```

Confirm compressed rotated logs are retained after later rotations:

```bash
ls -lh /var/log/httpd/*gz
```

Expected sanitized result:

```text
access_log.2.gz
error_log.2.gz
```

Confirm logrotate scheduling is active:

```bash
systemctl is-enabled logrotate.timer
systemctl is-active logrotate.timer
```

Expected sanitized result:

```text
enabled
active
```

## Outcome

Centralized log forwarding was configured and validated across Linux systems. Missing Graylog ingestion was traced to an inactive Syslog UDP input and corrected. Test messages and service-related events were confirmed through sanitized Graylog searches. HTTPD log rotation was configured for daily rotation with 14 retained logs, then validated through forced rotation and systemd timer checks.

## Lessons Learned

* Centralized logging should be validated end-to-end from client configuration to log search results.
* rsyslog forwarding changes require service restart and message-generation testing.
* Missing log events should be investigated across the full pipeline, including client configuration, firewall rules, listener ports, and Graylog input status.
* Graylog searches are useful for confirming service events across distributed Linux systems.
* Logrotate policies should define frequency, retention, compression, and service reload behavior.
* Forced log rotation is a useful validation step before relying on scheduled rotation.
