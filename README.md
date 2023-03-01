Ansible Role: ansible_role_adjoin
=========

Ansible role that joins supported Linux systems to an Active Directory. The role will modify krb5.conf, realmd.conf smb.conf, sssd.conf and any file in /etc/sssd/conf.d, /etc/krb5.conf.d. Any modifications will result in a backup of original file.
This role supports the following Linux distributions:

<ul>
<li>CentOS 7/8
<li>Fedora 34
<li>Red Hat Enterprise Linux 7/8
<li>Ubuntu 18/20/22
</ul>

Requirements
------------

Ansible:
This role has no dependencies on Ansible collections outside of Ansible Core.

Python:
pexpect >= 3.3 is needed for `ansible.builtin.expect` which is used during domain join process.


Role Variables
--------------

Available variables are listed below, along with default values where applicable (see `defaults/main.yml`):

| Variable | Required | Default | Comments |
| -------- | -------- | ------- | -------- |
| `ansible_role_adjoin_ad_access_filter` | No | | Access filter as per sssd-ad manpage, e.g. `DOM:test.domain.com:(memberOf:1.2.840.113556.1.4.1941:=CN=some_nested_group,OU=groups,OU=testing,DC=test,DC=domain,DC=com)` would allow access from anyone members/nested groups listed in the some_nested_group group, make note of that nested groups need the OID for LDAP_MATCHING_RULE_IN_CHAIN specified, as done through 1.2.840.113556.1.4.1941 in the above example. For a regular non group without support for nesting one could simply specify the following: `test.domain.com:(memberOf=CN=some_group,OU=groups,OU=testing,DC=test,DC=domain,DC=com)`. |
| `ansible_role_adjoin_cleanup` | No | true | Enable cleanup of any files in the .d folders of krb5 and sssd, default is true. |
| `ansible_role_adjoin_computer_ou` | No | | The full path to the OU where the computer account should be created. |
| `ansible_role_adjoin_domain` | Yes | | Name of the Active Directory domain that will be joined. |
| `ansible_role_adjoin_gpo_access_control` | No | permissive | Configures SSSD GPO-based accesscontrol. Default is permissive, which specifies that GPO-based access control is evaulated but not enforced. Valied values are `permissive`, `enforcing` and `disabled`. |
| `ansible_role_adjoin_hostname` | No | | Optionally specify an hostname to be set when running the role. |
| `ansible_role_adjoin_join_password` | Yes | | Password for the account used to join the system to Active Directory. |
| `ansible_role_adjoin_join_user` | Yes | | Username of the account used to join the system to Active Directory. |
| `ansible_role_adjoin_kdc` | Yes | | FQDN of the KDC of the domain. |
| `ansible_role_adjoin_renew_lifetime` | No | 7d | Renew time for the kerberos ticket. Default is 7 days. |
| `ansible_role_adjoin_sssd_services` | No | [] | A list of services that would work with SSSD. Default is nss and pam. |
| `ansible_role_adjoin_ticket_lifetime` | No | 24h | Lifetime of the kerberos ticket. |
| `ansible_role_adjoin_homedir_path` | No | /home/%f | Path to the homedir of the user, default is `/home/%f` which would result in /home/<username>@test.domain.com in the test.domain.com domain. |
| `ansible_role_adjoin_override_homedir` | No | true | Override the path to the homedir supplied by the Active directory, default is true. The path would be whatever path is specifed in ansible_role_adjoin_homedir_path. |
| `ansible_role_adjoin_ldap_user_extra_attrs` | No | [] | List of LDAP attributes that SSSD would fetch along with the usual set of user attributes, ex `altSecurityIdentities:altSecurityIdentities` if one would like to load SSH keys etc from altSecurityIdentities. |
| `ansible_role_adjoin_ldap_user_ssh_public_key` | No | | The LDAP attribute that contains the user's SSH public keys. |


Dependencies
------------

This role depends on https://github.com/mattias-jonsson/ansible_role_epel

Example Playbook
----------------

Joins the system to the test.domain.com domain and also loads SSH keys from the altSecurityIdentities LDAP attribute.

    - hosts: servers

      vars:
        ansible_role_adjoin_domain: test.domain.com
        ansible_role_adjoin_kdc: test.domain.com
        ansible_role_adjoin_ticket_lifetime: 24h
        ansible_role_adjoin_renew_lifetime: 7d
        ansible_role_adjoin_ad_access_filter: "DOM:test.domain.com:(memberOf:1.2.840.113556.1.4.1941:=CN=some_nested_group,OU=groups,OU=testing,DC=test,DC=domain,DC=com)"
        ansible_role_adjoin_computer_ou: cn=Computers,dc=test,dc=domain,dc=com
        ansible_role_adjoin_join_user: someuser
        ansible_role_adjoin_hostname: somehostname
        ansible_role_adjoin_join_password: somesecretpasssword
        ansible_role_adjoin_ldap_user_extra_attrs:
          - altSecurityIdentities:altSecurityIdentities
        ansible_role_adjoin_ldap_user_ssh_public_key: altSecurityIdentities
        ansible_role_adjoin_sssd_services:
          - nss
          - pam
          - ssh

      roles:
        - ansible_role_adjoin

License
-------

MIT

Author Information
------------------

Mattias Jonsson
