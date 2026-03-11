# LDAP Test Stack

This repository contains a minimal Docker-based LDAP test environment with:

- an LDAP server based on `rroemhild/test-openldap:latest`
- a phpLDAPadmin instance based on `osixia/phpldapadmin:0.9.0`
- sample LDIF files for directory initialization and test users

## Repository Layout

- `docker-compose.yml` - container definitions for LDAP and phpLDAPadmin
- `base.ldif` - sample base entry for `dc=example,dc=org`
- `user.ldif` - sample user entry under `ou=people,dc=planetexpress,dc=com`
- `newuser.ldif` - additional sample user entry under `ou=people,dc=planetexpress,dc=com`

## Services

### LDAP

- Container name: `test-openldap`
- Image: `rroemhild/test-openldap:latest`
- Host port: `1389`
- Container port: `10389`
- Connection URL: `ldap://localhost:1389`

### phpLDAPadmin

- Container name: `test-phpldapadmin`
- Image: `osixia/phpldapadmin:0.9.0`
- Host port: `6443`
- Container port: `443`
- URL: `https://localhost:6443`
- LDAP host configuration: `PHPLDAPADMIN_LDAP_HOSTS=ldap`

## Prerequisites

- Docker Engine
- Docker Compose v2

## Start the Stack

```bash
docker compose up -d
```

Check container status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs ldap
docker compose logs ldapadmin
```

## Stop the Stack

```bash
docker compose down
```

To remove associated volumes as well:

```bash
docker compose down -v
```

## LDIF Files

### `base.ldif`

Creates the following base entry:

```ldif
dn: dc=example,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: Example Org
dc: example
```

### `user.ldif`

Creates a sample user:

```ldif
dn: uid=testuser,ou=people,dc=planetexpress,dc=com
uid: testuser
cn: Test User
sn: User
mail: testuser@planetexpress.com
userPassword: testpass123
```

### `newuser.ldif`

Creates another sample user:

```ldif
dn: uid=newuser,ou=people,dc=planetexpress,dc=com
uid: mike
cn: New User
sn: User
mail: mike@planetexpress.com
userPassword: N@iAfg9M
```

## Important Directory Structure Note

The LDIF files do not currently use the same directory tree:

- `base.ldif` uses `dc=example,dc=org`
- `user.ldif` and `newuser.ldif` use `dc=planetexpress,dc=com`

This means the user records cannot be imported successfully unless the target directory information tree already contains:

- `dc=planetexpress,dc=com`
- `ou=people,dc=planetexpress,dc=com`

Before importing user entries, either:

1. modify the user LDIF files to match the base DN you are using, or
2. create the required parent entries for `dc=planetexpress,dc=com`

## Import LDIF Data

Use `ldapadd` from the host or from a container that has LDAP client tools installed.

Example:

```bash
ldapadd -x -H ldap://localhost:1389 -D "cn=admin,<base_dn>" -W -f base.ldif
```

Import a user entry:

```bash
ldapadd -x -H ldap://localhost:1389 -D "cn=admin,<base_dn>" -W -f user.ldif
```

If the directory entry already exists, use `ldapmodify` or delete the existing entry before re-importing.

## Accessing phpLDAPadmin

Open:

```text
https://localhost:6443
```

Because the container commonly serves a self-signed certificate in local setups, the browser may show a certificate warning. Accept the warning manually if required.

## Validation

To verify that the LDAP port is exposed:

```bash
nc -z localhost 1389
```

To verify that the web interface is exposed:

```bash
nc -z localhost 6443
```

To inspect the effective Compose configuration:

```bash
docker compose config
```

## Current Compose Definition

```yaml
services:
  ldap:
    image: rroemhild/test-openldap:latest
    container_name: test-openldap
    ports:
      - "1389:10389"
  ldapadmin:
    image: osixia/phpldapadmin:0.9.0
    container_name: test-phpldapadmin
    environment:
      PHPLDAPADMIN_LDAP_HOSTS: "ldap"
    ports:
      - "6443:443"
    depends_on:
      - ldap
```
