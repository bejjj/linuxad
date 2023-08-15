# Ansible - Linux Active Directory Domain Join
This is a small role to join a Ubuntu or CentOS machine to a 
Microsoft Active Directory domain. This isn't by any means polished or 
finished; it's just something I've glued together over the years
to help manage a large estate of Linux servers, and might be useful to someone
else.

Common group mappings can be applied to limit which domain users can login, and
which users can use `sudo`. Host-specific group mappings can also be applied, to
do the same for a specific host only.

# Configuration
The role expects a configuration structure as follows:
```yaml
winbind_domain: corp.mycompany.co.uk
winbind_workgroup: MYCOMPANY
winbind_required_login_group: S-1-5-21-1111111111-2222222222-3333333333-1001

winbind_groupmap_admins:
  - { ntgroup: "ServerAdmins", unixgroup: "sudo",  domain_sid: "S-1-5-21-1111111111-2222222222-3333333333-1002" }

winbind_groupmap_admins_centos:
  - { ntgroup: "ServerAdmins", unixgroup: "wheel",  domain_sid: "S-1-5-21-1111111111-2222222222-3333333333-1002" }

winbind_groupmap_users:
  - { ntgroup: "ServerUsers",  unixgroup: "staff", domain_sid: "S-1-5-21-1111111111-2222222222-3333333333-1001" }

winbind_groupmap_users_centos:
  - { ntgroup: "ServerUsers",  unixgroup: "users", domain_sid: "S-1-5-21-1111111111-2222222222-3333333333-1001" }

winbind_krb:
  realms:
    kdc:
      location1:
        - dc-location1.corp.mycompany.co.uk
        - dc-location2.corp.mycompany.co.uk
      location2:
        - dc-location2.corp.mycompany.co.uk
        - dc-location1.corp.mycompany.co.uk

    admin_server: corp.mycompany.co.uk
    default_domain: corp.mycompany.co.uk
  domain_realms:
    - .corp.mycompany.co.uk
    - corp.mycompany.co.uk

winbind_samba_conf:
  - { option: "security",  value: "ADS" }
  - { option: "workgroup", value: "{{ winbind_workgroup }}" }
  - { option: "realm",     value: "{{ winbind_domain    }}" }
  - { option: "allow dns updates", value: "disabled"  }
  - { option: "template shell",    value: "/bin/bash" }
  - { option: "template homedir",  value: "/home/%U"  }
  - { option: "idmap config * : range",     value: "200000-500000" }
  - { option: "winbind separator",          value: "+"       }
  - { option: "winbind use default domain", value: "yes"     }
  - { option: "winbind enum users",         value: "yes"     }
  - { option: "winbind enum groups",        value: "no"      }
  - { option: "winbind nss info",           value: "rfc2307" }
```
You will of course need to customise the domain, domain controllers, realms/sites, etc, and the group names and SIDs to suit.
The SID for a group can be obtained with a bit of PowerShell, using the `ActiveDirectory` module:
```
Get-ADGroup -Identity ServerAdmins | Select-Object Name, SID
```
You will also need to provide credentials with sufficient permission to join the domain:
```yaml
ad_domain_join:
    user: domain-join-user
    password: ***
```
Specific host group mappings can be set as follows:
```yaml
host_ad_groupmaps:
  - { ntgroup: "Server1-Users",  unixgroup: "users", domain_sid: "S-1-5-21-1111111111-2222222222-3333333333-1003" }
  - { ntgroup: "Server1-Admins",  unixgroup: "localadmins", domain_sid: "S-1-5-21-1111111111-2222222222-3333333333-1004" }
```
Placement of these will depend upon how you structure your shared and host-specific variables within your inventory.

`host_location_code` is expected to be defined in the inventory against the host, and is required to index into the available
domain locations, per the configuration above.

# Testing
This role has been used successfully on servers running Ubuntu 20.04 LTS, Ubuntu 22.04 LTS and CentOS 7.

# Usage
Build this role into a suitable playbook, add the configuration into your inventory, and use as normal.
I haven't provided an example playbook or inventory here, as it's likely you already have some structure around this already if you're joining Linux servers to an AD domain.