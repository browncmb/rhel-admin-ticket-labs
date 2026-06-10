# SSH Hardening and Bastion-Only Access

## Objective

Harden SSH access across Linux servers by enforcing key-based administrative access, disabling direct root SSH login, restricting inbound SSH to a bastion host, and validating the approved access path through troubleshooting and connection testing.

## Environment

* Red Hat-based Linux virtual machines
* OpenSSH server
* firewalld
* Bastion host access model
* Administrative Linux user with sudo privileges
* Sanitized hostnames, IP addresses, usernames, domains, and access paths

## Work Performed

* Verified administrative SSH access to managed Linux servers using a non-root user.
* Configured key-based SSH authentication for administrative access between trusted systems.
* Disabled direct SSH login for the root account.
* Validated SSH daemon configuration before applying changes.
* Restarted the SSH service to enforce the updated configuration.
* Restricted inbound SSH access using firewalld so managed servers accepted SSH only from the bastion host.
* Removed broad SSH access from the default firewall zone.
* Validated that SSH access succeeded through the bastion path.
* Confirmed that direct SSH access from non-approved sources was blocked.
* Investigated user-reported SSH failures and identified the bastion-only access control as the expected cause.

## Commands or Configuration

Generate an SSH key pair for an administrative user:

```bash
ssh-keygen -t ed25519 -C "linux-admin-access"
```

Copy the public key to an approved administrative host:

```bash
ssh-copy-id <admin_user>@<managed_server>
```

Verify non-root SSH access:

```bash
ssh <admin_user>@<managed_server>
```

Disable direct root SSH login using an SSH daemon drop-in file:

```bash
sudo vi /etc/ssh/sshd_config.d/10-disable-root-login.conf
```

```sshconfig
PermitRootLogin no
```

Validate SSH daemon syntax before restarting the service:

```bash
sudo sshd -t
```

Restart SSH to apply the hardening change:

```bash
sudo systemctl restart sshd
sudo systemctl status sshd --no-pager
```

Allow SSH only from the bastion host:

```bash
sudo firewall-cmd --permanent --zone=public \
  --add-rich-rule='rule family="ipv4" source address="<bastion_ip>/32" port port="22" protocol="tcp" accept'
```

Remove broad/default SSH exposure from the public zone:

```bash
sudo firewall-cmd --permanent --zone=public --remove-service=ssh
sudo firewall-cmd --permanent --zone=public --remove-port=22/tcp
sudo firewall-cmd --reload
```

Review the active firewall configuration:

```bash
sudo firewall-cmd --list-all
```

## Validation

Confirm direct root SSH login is denied:

```bash
ssh root@<managed_server>
```

Expected sanitized result:

```text
Permission denied.
```

Confirm SSH access succeeds from the bastion host:

```bash
ssh <admin_user>@<bastion_host>
ssh <admin_user>@<managed_server>
```

Confirm direct SSH access from a non-approved source is blocked:

```bash
ssh <admin_user>@<managed_server>
```

Expected sanitized result:

```text
Connection timed out
```

Validate the firewall only allows SSH from the approved bastion source:

```bash
sudo firewall-cmd --list-rich-rules
```

Expected sanitized result:

```text
rule family="ipv4" source address="<bastion_ip>/32" port port="22" protocol="tcp" accept
```

## Outcome

SSH access was hardened by disabling direct root login and limiting inbound SSH access to a bastion-controlled path. Administrative users retained access through the approved bastion workflow, while direct SSH attempts from non-approved sources were blocked. A later SSH access issue was traced to the expected behavior of the bastion-only restriction and resolved through user guidance on the correct access path.

## Lessons Learned

* SSH hardening should preserve administrative access while reducing unnecessary exposure.
* Root SSH login should be disabled in favor of named administrative users with sudo privileges.
* Bastion hosts provide a controlled access point for private Linux systems.
* Firewall changes should be validated from both approved and non-approved sources.
* User-reported access issues should be reviewed against recent security changes before treating them as service failures.
* Access-control changes should be documented clearly so administrators and users understand the approved connection path.
