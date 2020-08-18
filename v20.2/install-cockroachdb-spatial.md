---
title: Install CockroachDB Spatial
summary: Install instructions for CockroachDB with support for efficiently storing and querying spatial data.
toc: true
---

<a name="linux"></a>
<a name="mac"></a>
<span class="version-tag">New in v20.2</span>: CockroachDB has special support for efficiently storing and querying spatial data.

This page has instructions for installing CockroachDB Spatial on Mac and Linux.

If you are looking to install a single-file CockroachDB binary without spatial support, see [Install CockroachDB](install-cockroachdb.html).

{{site.data.alerts.callout_info}}
These instructions are likely to change, since they refer to an alpha build of CockroachDB. Note that the Windows installation instructions for CockroachDB Spatial are still being developed.

For instructions showing how to get started using CockroachDB Spatial, see [Working with Spatial Data](spatial-data.html).
{{site.data.alerts.end}}

<div class="filters clearfix">
  <button class="filter-button" data-scope="mac">Mac</button>
  <button class="filter-button" data-scope="linux">Linux</button>
</div>

<section class="filter-content" markdown="1" data-scope="mac">

## Mac installation

### Step 1. Download the `cockroach` binary and supporting spatial libraries

{% include copy-clipboard.html %}
~~~ shell
wget https://edge-binaries.cockroachdb.com/cockroach/cockroach.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2
~~~

{% include copy-clipboard.html %}
~~~ shell
wget https://edge-binaries.cockroachdb.com/cockroach/lib/libgeos.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.dylib
~~~

{% include copy-clipboard.html %}
~~~ shell
wget https://edge-binaries.cockroachdb.com/cockroach/lib/libgeos_c.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.dylib
~~~

### Step 2. Copy libraries to the appropriate locations

First, rename the libraries to remove the build SHA:

{% include copy-clipboard.html %}
~~~ shell
mv libgeos.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.dylib libgeos.darwin-amd64.dylib
~~~

{% include copy-clipboard.html %}
~~~ shell
mv libgeos_c.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.dylib libgeos_c.darwin-amd64.dylib
~~~

Next, copy the libraries to the location where CockroachDB expects to find them (by default this is `/usr/local/lib/cockroach` but you can customize the location using the `--geo-libs` flag to [`cockroach start`](cockroach-start.html) if you are an expert):

{% include copy-clipboard.html %}
~~~ shell
mkdir -p /usr/local/lib/cockroach
~~~

{% include copy-clipboard.html %}
~~~ shell
cp libgeos.darwin-amd64.dylib /usr/local/lib/cockroach
~~~

{% include copy-clipboard.html %}
~~~ shell
cp libgeos_c.darwin-amd64.dylib /usr/local/lib/cockroach
~~~

### Step 3. Copy the binary to an appropriate location

Like the libraries, the `cockroach` binary needs to be renamed to remove the build SHA:

{% include copy-clipboard.html %}
~~~ shell
mv cockroach.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2 cockroach
~~~

The `cockroach` binary needs to go somewhere on your shell's `PATH`. A [standard location](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s09.html) for user-installed binaries is `/usr/local/bin`. If you are an expert, you can install `cockroach` somewhere else.

{% include copy-clipboard.html %}
~~~ shell
mv cockroach /usr/local/bin
~~~

#### Step 4. Test the installation

To make sure that CockroachDB Spatial is properly installed and can execute spatial queries, do the steps listed below.

First, make sure the `cockroach` binary we just installed is the one that runs when you type `cockroach` in your shell:

{% include copy-clipboard.html %}
~~~ shell
which cockroach
~~~

~~~
/usr/local/bin/cockroach
~~~

Next, start the `cockroach` binary using [`cockroach start`](cockroach-start.html):

{% include copy-clipboard.html %}
~~~ shell
cockroach start-single-node --insecure
~~~

This should generate output that looks like the following:

~~~
*
* WARNING: RUNNING IN INSECURE MODE!
* 
* - Your cluster is open for any client that can access <all your IP addresses>.
* - Any user, even root, can log in without providing a password.
* - Any user, connecting as root, can read or write any data in your cluster.
* - There is no network encryption nor authentication, and thus no confidentiality.
* 
* Check out how to secure your cluster: https://www.cockroachlabs.com/docs/v20.2/secure-a-cluster.html
*
*
* WARNING: neither --listen-addr nor --advertise-addr was specified.
* The server will advertise "Richards-MBP" to other nodes, is this routable?
* 
* Consider using:
* - for local-only servers:  --listen-addr=localhost
* - for multi-node clusters: --advertise-addr=<host/IP addr>
* 
*
*
* INFO: Replication was disabled for this cluster.
* When/if adding nodes in the future, update zone configurations to increase the replication factor.
*
CockroachDB node starting at 2020-08-18 18:10:16.218622 +0000 UTC (took 1.5s)
build:               CCL v20.2.0-alpha.1-2192-g6ff42f2b29 @ 2020/08/14 18:55:52 (go1.13.14)
webui:               http://Richards-MBP:8080
sql:                 postgresql://root@Richards-MBP:26257?sslmode=disable
RPC client flags:    /usr/local/bin/cockroach <client cmd> --host=Richards-MBP:26257 --insecure
logs:                /Users/rloveland/cockroach-data/logs
temp dir:            /Users/rloveland/cockroach-data/cockroach-temp921402564
external I/O path:   /Users/rloveland/cockroach-data/extern
store[0]:            path=/Users/rloveland/cockroach-data
storage engine:      pebble
status:              initialized new cluster
clusterID:           d47c1b65-1fd2-457b-9a66-86096b6392aa
nodeID:              1
~~~

Next, to test that the spatial libraries have loaded properly, run the following query:

{% include copy-clipboard.html %}
~~~ sql
SELECT ST_IsValid(ST_MakePoint(1,2));
~~~

This should result in the following output. If it does not, your `cockroach` binary is not properly accessing the dynamically linked C libraries in `/usr/local/lib/cockroach` for some reason. If you are having a hard time installing CockroachDB Spatial, please see our [Support Resources](support-resources.html).

~~~
  st_isvalid
--------------
     true
(1 row)
~~~

</section>

<section class="filter-content" markdown="1" data-scope="linux">

## Linux installation

### Step 1. Download the `cockroach` binary and supporting spatial libraries

{% include copy-clipboard.html %}
~~~ shell
wget https://edge-binaries.cockroachdb.com/cockroach/cockroach.linux-gnu-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2
~~~

{% include copy-clipboard.html %}
~~~ shell
wget https://edge-binaries.cockroachdb.com/cockroach/lib/libgeos.linux-gnu-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.so
~~~

{% include copy-clipboard.html %}
~~~ shell
wget https://edge-binaries.cockroachdb.com/cockroach/lib/libgeos_c.linux-gnu-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.so
~~~

### Step 2. Copy libraries to the appropriate locations

First, rename the libraries to remove the build SHA:

{% include copy-clipboard.html %}
~~~ shell
mv libgeos.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.so libgeos.so
~~~

{% include copy-clipboard.html %}
~~~ shell
mv libgeos_c.darwin-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2.so libgeos_c.so
~~~

Next, copy the libraries to the location where CockroachDB expects to find them (by default this is `/usr/local/lib/cockroach` but you can customize the location using the `--geo-libs` flag to [`cockroach start`](cockroach-start.html) if you are an expert):

{% include copy-clipboard.html %}
~~~ shell
mkdir -p /usr/local/lib/cockroach
~~~

{% include copy-clipboard.html %}
~~~ shell
cp libgeos.so /usr/local/lib/cockroach
~~~

{% include copy-clipboard.html %}
~~~ shell
cp libgeos_c.so /usr/local/lib/cockroach
~~~

### Step 3. Copy the binary to an appropriate location

Like the libraries, the `cockroach` binary needs to be renamed to remove the build SHA:

{% include copy-clipboard.html %}
~~~ shell
mv cockroach.linux-gnu-amd64.6ff42f2b29c668bc474a73d8f7b50f9fd45c1cb2 cockroach
~~~

The `cockroach` binary needs to go somewhere on your shell's `PATH`. A [standard location](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s09.html) for user-installed binaries is `/usr/local/bin`. If you are an expert, you can install `cockroach` somewhere else.

{% include copy-clipboard.html %}
~~~ shell
mv cockroach /usr/local/bin
~~~

### Step 4. Test the installation

To make sure that CockroachDB Spatial is properly installed and can execute spatial queries, do the steps listed below.

First, make sure the `cockroach` binary we just installed is the one that runs when you type `cockroach` in your shell:

{% include copy-clipboard.html %}
~~~ shell
which cockroach
~~~

~~~
/usr/local/bin/cockroach
~~~

Next, start the `cockroach` binary using [`cockroach start`](cockroach-start.html):

{% include copy-clipboard.html %}
~~~ shell
cockroach start-single-node --insecure
~~~

This should generate output that looks like the following:

~~~
*
* WARNING: RUNNING IN INSECURE MODE!
* 
* - Your cluster is open for any client that can access <all your IP addresses>.
* - Any user, even root, can log in without providing a password.
* - Any user, connecting as root, can read or write any data in your cluster.
* - There is no network encryption nor authentication, and thus no confidentiality.
* 
* Check out how to secure your cluster: https://www.cockroachlabs.com/docs/v20.2/secure-a-cluster.html
*
*
* WARNING: neither --listen-addr nor --advertise-addr was specified.
* The server will advertise "Richards-MBP" to other nodes, is this routable?
* 
* Consider using:
* - for local-only servers:  --listen-addr=localhost
* - for multi-node clusters: --advertise-addr=<host/IP addr>
* 
*
*
* INFO: Replication was disabled for this cluster.
* When/if adding nodes in the future, update zone configurations to increase the replication factor.
*
CockroachDB node starting at 2020-08-18 18:10:16.218622 +0000 UTC (took 1.5s)
build:               CCL v20.2.0-alpha.1-2192-g6ff42f2b29 @ 2020/08/14 18:55:52 (go1.13.14)
webui:               http://localhost:8080
sql:                 postgresql://root@localhost:26257?sslmode=disable
RPC client flags:    /usr/local/bin/cockroach <client cmd> --host=localhost:26257 --insecure
logs:                /Users/rloveland/cockroach-data/logs
temp dir:            /Users/rloveland/cockroach-data/cockroach-temp921402564
external I/O path:   /Users/rloveland/cockroach-data/extern
store[0]:            path=/Users/rloveland/cockroach-data
storage engine:      pebble
status:              initialized new cluster
clusterID:           d47c1b65-1fd2-457b-9a66-86096b6392aa
nodeID:              1
~~~

Next, to test that the spatial libraries have loaded properly, run the following query:

{% include copy-clipboard.html %}
~~~ sql
SELECT ST_IsValid(ST_MakePoint(1,2));
~~~

This should result in the following output. If it does not, your `cockroach` binary is not properly accessing the dynamically linked C libraries in `/usr/local/lib/cockroach` for some reason. If you are having a hard time installing CockroachDB Spatial, please see our [Support Resources](support-resources.html).

~~~
  st_isvalid
--------------
     true
(1 row)
~~~

</section>

## See also

- [Working with Spatial Data](spatial-data.html)
- [Spatial Features](spatial-features.html)
- [Spatial & GIS Glossary of Terms](spatial-glossary.html)
- [Geospatial functions](functions-and-operators.html#geospatial-functions)
