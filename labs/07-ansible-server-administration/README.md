# Ansible Server Administration and Firewall Automation

## Objective

Use Ansible from a centralized control node to validate Linux managed-node connectivity, troubleshoot privilege escalation, apply server administration changes, and automate firewalld configuration for managed RHEL-based systems.

## Environment

* Red Hat-based Linux virtual machines
* Ansible control node
* Managed Linux servers
* SSH key-based authentication
* sudo-based privilege escalation
* firewalld

## Work Performed

* Prepared an Ansible control node for Linux server administration.
* Configured SSH key-based access from the control node to managed Linux servers.
* Created a sanitized Ansible inventory with grouped managed nodes.
* Validated managed-node connectivity using the Ansible ping module.
* Ran ad hoc Ansible commands to confirm remote command execution.
* Reviewed privilege escalation behavior during administrative tasks.
* Troubleshot a failed Ansible run caused by missing sudo authentication.
* Updated the playbook execution workflow to use privilege escalation correctly.
* Applied server administration changes through Ansible.
* Automated firewalld changes to close unnecessary HTTP and HTTPS exposure.
* Validated the firewall state after the playbook completed.

## Commands and Configuration

Example sanitized inventory:

```ini
[dev_servers]
app01.example.lab
web01.example.lab

[linux_servers:children]
dev_servers
```

Validate Ansible inventory parsing:

```bash
ansible-inventory -i inventory/hosts.ini --list
```

Test managed-node connectivity:

```bash
ansible dev_servers -i inventory/hosts.ini -m ping
```

Run an ad hoc command against managed nodes:

```bash
ansible dev_servers -i inventory/hosts.ini -m command -a "hostname"
```

Example server administration playbook structure:

```yaml
---
- name: Apply baseline administration tasks
  hosts: dev_servers
  become: true

  tasks:
    - name: Ensure selected packages are current
      ansible.builtin.dnf:
        name: "*"
        state: latest
```

Example firewalld automation to close HTTP and HTTPS exposure:

```yaml
---
- name: Remove unnecessary web service exposure
  hosts: dev_servers
  become: true

  tasks:
    - name: Remove HTTP and HTTPS services from the active firewall configuration
      ansible.posix.firewalld:
        service: "{{ firewall_service }}"
        permanent: true
        immediate: true
        state: disabled
      loop:
        - http
        - https
      loop_control:
        loop_var: firewall_service
        label: "{{ firewall_service }}"
```

Validate playbook syntax before execution:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/manage-firewall.yml --syntax-check
```

Run the playbook:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/manage-firewall.yml
```

Run with privilege escalation prompting when required:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/manage-firewall.yml --ask-become-pass
```

## Validation

Confirm managed nodes respond to Ansible:

```bash
ansible dev_servers -i inventory/hosts.ini -m ping
```

Expected sanitized result:

```text
app01.example.lab | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Confirm playbook syntax validation succeeds:

```text
playbook: playbooks/manage-firewall.yml
```

Confirm the playbook completes without failed hosts:

```text
PLAY RECAP
app01.example.lab : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
web01.example.lab : ok=3 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

Confirm HTTP and HTTPS are no longer enabled in firewalld:

```bash
ansible dev_servers -i inventory/hosts.ini -m command -a "firewall-cmd --list-services" --become
```

Expected sanitized result:

```text
dhcpv6-client ssh
```

## Outcome

Ansible was used to administer multiple Linux servers from a centralized control node. Managed-node connectivity was validated, privilege escalation issues were identified and corrected, and firewall configuration changes were automated and verified. The resulting workflow demonstrated repeatable Linux administration through inventory targeting, playbook execution, sudo escalation, and post-change validation.

## Lessons Learned

* Ansible connectivity should be validated before running administrative playbooks.
* SSH key-based access improves automation reliability when paired with controlled sudo privileges.
* Privilege escalation failures should be reviewed before assuming a playbook or module issue.
* Syntax checks and targeted validation reduce risk before applying configuration changes.
* firewalld changes should be confirmed after automation runs to verify the intended service exposure.
* Clean inventory grouping makes operational targeting easier to understand and maintain.
