# Test node-oracledb

*Copyright (c) 2015, 2018, Oracle and/or its affiliates. All rights reserved.*

You may not use the identified files except in compliance with the Apache
License, Version 2.0 (the "License.")

You may obtain a copy of the License at
http://www.apache.org/licenses/LICENSE-2.0.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

See the License for the specific language governing permissions and
limitations under the License.

The node-oracledb test suite uses 'mocha', 'should' and 'async'.  See
[LICENSE](https://github.com/oracle/node-oracledb/blob/master/LICENSE.md)
for relevant licenses.

## 1. Preparations

See [INSTALL](https://oracle.github.io/node-oracledb/INSTALL.html)
for installation details.

Note: the
[test suite](https://github.com/oracle/node-oracledb/tree/master/test)
is on GitHub. NPM module has not contained the tests since node-oracledb 1.9.1.

### 1.1 Create a working directory

```
mkdir <some-directory>
cd <some-directory>
```

### 1.2 Clone node-oracledb from GitHub

Clone the project repository:

```
cd <some-directory>
git clone https://github.com/oracle/node-oracledb.git
```

### 1.3 Build

```
cd <some-directory>/node-oracledb
npm install
```

Running `npm install` within the node-oracledb/ directory will recompile
oracledb and install all its dependent modules.  These are listed
in the `devDependencies` field of `package.json` file.  Thus, 'mocha', 'async'
and 'should' modules are installed by this command.

The test suite uses [mocha](https://www.npmjs.com/package/mocha),
[async](https://www.npmjs.com/package/async) and
[should](https://www.npmjs.com/package/should).

### 1.4 Configure Database credentials

The file `test\dbconfig.js` contains the configuration. Change the credential information to yours.

To enable external authentication tests, firstly make sure the Oracle external authentication service is correctly configured.
See [Documentation for External Authentication](https://oracle.github.io/node-oracledb/doc/api.html#extauth) for details.

Note: the test suite requires a schema with privileges CREATE TABLE, CREATE SESSION,
CREATE PROCEDURE, CREATE SEQUENCE, CREATE TRIGGER.

### 1.5 Set NODE_PATH

```bash
export NODE_PATH=<some-directory>/node-oracledb/lib
```

### 1.6 Available Environment Variables

Setting the following environment variables will override the default values defined in test/dbconfig.js.

* `NODE_ORACLEDB_USER` overrides the username of the schema user which you used for testing.

* `NODE_ORACLEDB_PASSWORD` overrides the password of the schema user which you used for testing.

* `NODE_ORACLEDB_CONNECTIONSTRING` overrides the connection string that points to your database's location.

* `NODE_ORACLEDB_EXTERNALAUTH` overrides the options for external authentication tests. Setting this environment variable to "true" will enable the tests that require external authentication.

* `NODE_ORACLEDB_DBA_PRIVILEGE` overrides the options for DBA privilege. Setting this environment variable to "true" will enable the tests and utilities that require DBA privilege.

* `NODE_ORACLEDB_DBA_USER` overrides the username of the DBA user which you used for testing, disabled if `NODE_ORACLEDB_DBA_PRIVILEGE` is not `true`.

* `NODE_ORACLEDB_DBA_PASSWORD` overrides the password of the DBA user which you used for testing, disabled if `NODE_ORACLEDB_DBA_PRIVILEGE` is not `true`.

* `NODE_ORACLEDB_PROXY_SESSION_USER` provides the username of a schema user that can connect through the schema user which you used for testing using proxy authentication. Setting this environment variable will enable the tests that require proxy authentication.


## 2. Run tests

### 2.1 Run the complete test suite

```
cd node-oracledb
npm test
```

### 2.2 Run specified test(s)

```
cd node_oracledb
./node_modules/.bin/mocha test/<test-names>
```

See [mochajs.org](http://mochajs.org/) for more information on running tests with mocha.

## 3. Enable test(s) that requires extra configuration

The following test(s) are automatically skipped if their required environment variable(s) are not properly set.

### 3.1 externalProxyAuth.js
This test aims to test the combined usage of external authentication and proxy authentication. To run this test, you need to complete the following prerequisite setups.

* Enable external authentication on the schema user which you used for testing. See [Documentation for External Authentication](https://oracle.github.io/node-oracledb/doc/api.html#extauth) for more information on external authentication. Then use the following command to enable external authentication in the test suite.

	```
	export NODE_ORACLEDB_EXTERNALAUTH true

	```

* Enable proxy authentication on another schema user specified by environment variable `NODE_ORACLEDB_PROXY_SESSION_USER` that connects through the schema user which you used for testing. See [Documentation for Pool Proxy Authentication](https://oracle.github.io/node-oracledb/doc/api.html#pool-proxy-authentication) for more information on proxy authentication. Then use the following command to enable proxy authentication in the test suite.

	```
	export NODE_ORACLEDB_PROXY_SESSION_USER "Your_Proxy_Authenticating_User"

	```


## 4. Add Tests

See [CONTRIBUTING](https://github.com/oracle/node-oracledb/blob/master/CONTRIBUTING.md)
for general information on contribution requirements.

For easy correlation between results and test code, each test is
assigned a number. The [Test List](https://github.com/oracle/node-oracledb/blob/master/test/list.txt)
shows the numbering of tests.

In order to include your tests in the suite, add each new test file
name to [`test/opts/mocha.opts`](https://github.com/oracle/node-oracledb/blob/master/test/opts/mocha.opts).

## 5. Troubleshooting

You may encounter some troubles when running the test suite. These troubles
might be caused by the concurrency issue of Mocha framework, network latencies,
or database server issues. This section gives some issues that we ever saw
and our solutions.

### 5.1 ORA-00054: resource busy and acquire with NOWAIT specified or timeout expired

This error occurs when Node.js programs try to change database objects which
hold locks. The workaround would be:

(1) Use unique DB object names for each test to avoid interference between
test files.
(2) Try not to use 'beforeEach' blocks for object operations to avoid
the interference between cases.

### 5.2 ORA-00018: maximum number of sessions exceeded

This error occurs when the test suite takes up more sessions than the
configured limit. You can alter the session limit on the database server side.
If you do not have access to change the database session setting, you could
use the below script to deliberately add an interval between tests.

```Bash
arr=$(ls test/*js)
for case in ${arr[@]}
do
  var="$NODE_PATH/../node_modules/.bin/mocha --timeout 10000 $case"
  eval $var
  sleep 1
done
```

### 5.3 ORA-28865: SSL connection closed

You may encounter this error when the test suite sends more connection
requests per second than the database is configured to handle.

There are two solutions:

- Solution 1: Change database `RATE_LIMIT` configuration. This parameter
defines the connection count allowed per second. See [RATE_LIMIT](https://www.oracle.com/pls/topic/lookup?ctx=dblatest&id=GUID-F302BF91-64F2-4CE8-A3C7-9FDB5BA6DCF8)
for more information.

- Solution 2: Set the `RETRY_COUNT` and `RETRY_DELAY` parameters in
connectString.

For example, below is the connectString which could be defined in
`tnsnames.ora` file.

```
dbaccess = (description=(RETRY_COUNT=20)(RETRY_DELAY=3)
          (address=(protocol=tcps)(port=1521)(host=<db-host>))
          (connect_data=(service_name=<service-name>))
          (security=(my_wallet_directory=<wallet-location>)(ssl_server_cert_dn=<ssl-server-cert-dn>))
       )
```
