To run tests use supplied Dockerfile.test:

```shell
docker build -t my-tag -f Dockerfile.test .
```

If you desire to use a container with Python3, you can supply an appropriate
build argument:

```shell
docker build -f Dockerfile.test -t my-tag --build-arg PYTHON_VERSION=3 .
docker run my-tag
```

To run without Docker:

Test suite is available at http://hg.nginx.org/nginx-tests.
Check the http://hg.nginx.org/nginx-tests/file/tip/README file
for instructions on how to use it.

Additionally, the test requires a working installation
of OpenLDAP server and utilities (http://www.openldap.org/),
and python's coverage tool (https://coverage.readthedocs.io)

copy ldap-auth.t into testsuite, setup environment variables:

$ export TEST_LDAP_DAEMON=/usr/lib64/openldap/slapd
$ export TEST_LDAP_AUTH_DAEMON=/path/to/nginx-ldap-auth-daemon.py
$ prove 'ldap-auth.t'

to get coverage report:

$ export TEST_NGINX_LEAVE=1
$ prove 'ldap-auth.t'
$ cd /tmp/nginx-test-xxxx
$ coverage2 html

report is now generated in htmlcov/
