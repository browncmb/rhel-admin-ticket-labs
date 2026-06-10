# FreeIPA Client Enrollment and Identity Validation

## Objective

Enroll RHEL-based Linux systems into a centralized FreeIPA identity domain, validate SSSD and Kerberos authentication, confirm centralized user and group lookup, and verify domain-based access behavior.

## Environment

* Red Hat-based Linux virtual machines
* FreeIPA server
* FreeIPA client packages
* SSSD
* Kerberos
* DNS and hostname prerequisites
* Centralized user and group management
* sudo/access validation

## Work Performed

* Verified hostname and name-resolution prerequisites before client enrollment.
* Installed required FreeIPA client packages on Linux systems.
* Enrolled Linux clients into a centralized FreeIPA identity domain.
* Used forced rejoin behavior only when a host enrollment already existed.
* Enabled home directory creation during client enrollment.
* Confirmed SSSD configuration and service state after enrollment.
* Validated centralized identity lookup with `id` and `getent`.
* Used Kerberos authentication before running identity administration queries.
* Verified Kerberos ticket presence with `klist`.
* Confirmed domain user and group membership visibility from Linux clients.
* Reviewed sudo and access behavior for domain-based users.
* Cleared SSSD cache or restarted SSSD when identity data required refresh.

## Commands and Configuration

Install FreeIPA client packages:

```bash
sudo dnf install -y ipa-client sssd oddjob-mkhomedir
```

Verify hostname configuration before enrollment:

```bash
hostnamectl
```

Validate name resolution for the FreeIPA server:

```bash
getent hosts <ipa_server_fqdn>
```

Enroll the Linux client into the sanitized FreeIPA domain:

```bash
sudo ipa-client-install \
  --domain=<ipa_domain> \
  --realm=<IPA_REALM> \
  --server=<ipa_server_fqdn> \
  --mkhomedir
```

Rejoin an existing sanitized host enrollment only when required:

```bash
sudo ipa-client-install \
  --domain=<ipa_domain> \
  --realm=<IPA_REALM> \
  --server=<ipa_server_fqdn> \
  --mkhomedir \
  --force-join
```

Validate SSSD service state:

```bash
systemctl status sssd --no-pager
systemctl is-active sssd
systemctl is-enabled sssd
```

Review the active authentication profile:

```bash
authselect current
```

Validate centralized user lookup:

```bash
id <domain_user>
getent passwd <domain_user>
```

Validate centralized group lookup:

```bash
getent group <domain_group>
```

Request a Kerberos ticket:

```bash
kinit <domain_user>
```

Verify Kerberos ticket status:

```bash
klist
```

Run a sanitized FreeIPA identity query:

```bash
ipa user-show <domain_user>
```

Clear SSSD cache if identity data needs to be refreshed:

```bash
sudo sss_cache -E
sudo systemctl restart sssd
```

Validate domain user sudo or access behavior:

```bash
su - <domain_user>
sudo -l
```

## Validation

Confirm FreeIPA server name resolution:

```bash
getent hosts <ipa_server_fqdn>
```

Expected sanitized result:

```text
<ipa_server_ip> <ipa_server_fqdn> <ipa_server_shortname>
```

Confirm SSSD is active:

```bash
systemctl is-active sssd
```

Expected sanitized result:

```text
active
```

Confirm SSSD is enabled at boot:

```bash
systemctl is-enabled sssd
```

Expected sanitized result:

```text
enabled
```

Confirm centralized user lookup:

```bash
id <domain_user>
```

Expected sanitized result:

```text
uid=<uid>(<domain_user>) gid=<gid>(<domain_group>) groups=<gid>(<domain_group>),<gid>(<additional_group>)
```

Confirm centralized account resolution:

```bash
getent passwd <domain_user>
```

Expected sanitized result:

```text
<domain_user>:*:<uid>:<gid>:<display_name>:/home/<domain_user>:/bin/bash
```

Confirm Kerberos authentication:

```bash
kinit <domain_user>
klist
```

Expected sanitized result:

```text
Default principal: <domain_user>@<IPA_REALM>
```

Confirm FreeIPA identity query works after Kerberos authentication:

```bash
ipa user-show <domain_user>
```

Expected sanitized result:

```text
User login: <domain_user>
First name: <first_name>
Last name: <last_name>
Account disabled: False
```

Confirm access behavior for the domain user:

```bash
su - <domain_user>
pwd
```

Expected sanitized result:

```text
/home/<domain_user>
```

## Outcome

Linux clients were enrolled into a centralized FreeIPA identity domain and validated through SSSD service checks, Kerberos authentication, centralized user and group lookup, and access testing. The completed workflow confirmed that identity services were available to managed Linux systems and that domain users could be resolved and authenticated as expected.

## Lessons Learned

* FreeIPA client enrollment depends on correct hostname and name-resolution prerequisites.
* SSSD is the key Linux integration layer for centralized identity lookup and authentication.
* Kerberos authentication should be validated before running identity administration commands.
* `id`, `getent`, `kinit`, and `klist` provide practical validation of identity integration.
* Existing host enrollments may require a controlled rejoin process rather than a new enrollment.
* Identity troubleshooting should include DNS, SSSD service state, Kerberos tickets, cache refresh, and access validation.
