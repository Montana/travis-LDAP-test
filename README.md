# Travis CI LDAP Tests 

## YAML config 

```yaml
language: php
sudo: false
addons:
  apt:
    packages:
    - ldap-utils
    - slapd
```

## Configuration

The LDAP server needs some configuration: The root DN, admin username and password (use `slappasswd`):

```bash
include   /etc/ldap/schema/core.schema
include   /etc/ldap/schema/cosine.schema
include   /etc/ldap/schema/inetorgperson.schema
include   /etc/ldap/schema/nis.schema
pidfile  /tmp/slapd/slapd.pid
argsfile /tmp/slapd/slapd.args
modulepath /usr/lib/openldap
database  ldif
directory /tmp/slapd
suffix    "dc=example,dc=com"
rootdn    "cn=admin,dc=example,dc=com"
rootpw    {SSHA}password
```
If you're not running on root, run this on port > `1024`, apart from the basic configuration, the unit tests require some test data to exist, so we have to import them before. PHP does not have the LDAP module loaded, so it ends up looking something like this:

```yaml
before_script:
  - phpenv config-add tests/travis/enable-ldap.ini
  - mkdir /tmp/slapd
  - slapd -f tests/ldif_data/slapd.conf -h ldap://localhost:3389 &
  - sleep 3
  - ldapadd -h localhost:3389 -D cn=admin,dc=example,dc=com -w test -f tests/ldif_data/base.ldif
  - ldapadd -h localhost:3389 -D cn=admin,dc=example,dc=com -w test -f tests/ldif_data/INITIAL_TESTDATA.ldif
```

If `pear` fails, try the following:

```yaml
language: php
sudo: false
php:
  - 7.4
addons:
  apt:
    packages:
    - ldap-utils
    - slapd
before_script:
  - phpenv config-add tests/travis/enable-ldap.ini
  - pear upgrade pear
  - cat tests/ldapconfig.ini.dist | sed s/389/3389/ > tests/ldapconfig.ini
  - mkdir /tmp/slapd
  - slapd -f tests/ldif_data/slapd.conf -h ldap://localhost:3389 &
  - sleep 3
  - ldapadd -h localhost:3389 -D cn=admin,dc=example,dc=com -w test -f tests/ldif_data/base.ldif
  - ldapadd -h localhost:3389 -D cn=admin,dc=example,dc=com -w test -f tests/ldif_data/INITIAL_TESTDATA.ldif

script:
  - cd tests
  - phpunit .
 ```
