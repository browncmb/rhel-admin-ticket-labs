# Apache Service Configuration with SELinux and firewalld

## Objective

Configure Apache HTTP Server on a RHEL-based system, serve content on a nonstandard TCP port, apply the required SELinux port labeling, allow access through firewalld, and validate HTTP service availability.

## Environment

* Red Hat-based Linux virtual machines
* Apache HTTP Server
* SELinux enforcing mode
* firewalld
* systemd-managed services
* Sanitized hostnames, IP addresses, usernames, URLs, and service output

## Work Performed

* Installed Apache HTTP Server on a Linux web server.
* Enabled and started the `httpd` service.
* Verified Apache service state with systemd.
* Deployed basic test content under the Apache document root.
* Configured Apache to listen on a nonstandard TCP port.
* Installed SELinux management utilities where required.
* Added the nonstandard Apache port to the SELinux `http_port_t` type.
* Opened the required TCP port with firewalld.
* Reloaded firewall configuration.
* Restarted Apache to apply configuration changes.
* Validated HTTP response behavior locally and remotely with `curl`.
* Confirmed service availability after SELinux and firewall changes.

## Commands and Configuration

Install Apache:

```bash
sudo dnf install -y httpd
```

Enable and start Apache:

```bash
sudo systemctl enable --now httpd
sudo systemctl status httpd --no-pager
```

Create basic test content:

```bash
echo "Apache validation page" | sudo tee /var/www/html/index.html
```

Configure Apache to listen on a nonstandard port:

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

Example sanitized Apache listener configuration:

```apache
Listen 8001
```

Install SELinux management utilities:

```bash
sudo dnf install -y policycoreutils-python-utils
```

Label the nonstandard port for Apache traffic:

```bash
sudo semanage port -a -t http_port_t -p tcp 8001
```

If the port already exists, modify the existing label:

```bash
sudo semanage port -m -t http_port_t -p tcp 8001
```

Verify SELinux port labeling:

```bash
sudo semanage port -l | grep http_port_t
```

Allow the Apache port through firewalld:

```bash
sudo firewall-cmd --permanent --add-port=8001/tcp
sudo firewall-cmd --reload
```

Restart Apache:

```bash
sudo systemctl restart httpd
```

## Validation

Confirm Apache is active:

```bash
systemctl is-active httpd
```

Expected sanitized result:

```text
active
```

Confirm Apache is enabled at boot:

```bash
systemctl is-enabled httpd
```

Expected sanitized result:

```text
enabled
```

Confirm SELinux recognizes the nonstandard Apache port:

```bash
sudo semanage port -l | grep http_port_t
```

Expected sanitized result:

```text
http_port_t    tcp    8001, 80, 443
```

Confirm firewalld allows the nonstandard Apache port:

```bash
sudo firewall-cmd --list-ports
```

Expected sanitized result:

```text
8001/tcp
```

Validate local HTTP response:

```bash
curl http://localhost:8001/
```

Expected sanitized result:

```text
Apache validation page
```

Validate remote HTTP response:

```bash
curl http://<web_server>:8001/
```

Expected sanitized result:

```text
Apache validation page
```

## Outcome

Apache HTTP Server was installed, enabled, configured to listen on a nonstandard TCP port, and validated successfully. SELinux was updated to allow Apache to bind to the custom port, firewalld was configured to permit traffic, and HTTP response testing confirmed the service was reachable as expected.

## Lessons Learned

* Apache service changes should be validated with both systemd and HTTP response testing.
* Nonstandard Apache ports require both firewall access and SELinux port labeling.
* SELinux can block otherwise correct service configurations if port contexts are not updated.
* firewalld changes should be reloaded and verified after port or service updates.
* `curl` provides a simple and reliable way to confirm local and remote web service behavior.
* Service configuration work is strongest when installation, access control, SELinux, firewall, and validation are documented together.
