
# Splunk Routing and Filtering Documentation

## Overview

Routing and filtering in Splunk allow you to direct specific events to particular indexers, forwarders, or destinations based on defined criteria. This is accomplished using several configuration files: `props.conf`, `transforms.conf`, and `outputs.conf`.

NOTE: Only Heavy Forwarders can route or filter all data based on events.

## Configuration Files

### props.conf

The `props.conf` file is used to define event processing rules for data inputs. It specifies how data should be parsed and processed.

-  **Source Type**: The type of data source. This can be "sourcetype" or "host".
-  **Field Extraction**: Defines how fields are extracted from events.

**Example:**

```ini
[my_sourcetype]
TRANSFORMS-set = <transforms_stanza_name>
```

### transforms.conf

The `transforms.conf` file is used in conjunction with `props.conf` to define transformation rules. These rules can include event routing, filtering, and field manipulation.

- **REGEX**: The regular expression used to match events.
- **DEST_KEY**: The destination key, such as `_TCP_ROUTING`.
- **FORMAT**: The format of the transformation. This will usually be the tcpout stanza name.

**Example:**

```ini
[transforms_stanza_name]
REGEX = . # This would select everything
DEST_KEY = _TCP_ROUTING
FORMAT = <target_group> 	#This will be what what you decide to name your group in outputs.conf 
```
### outputs.conf

The `outputs.conf` file is used to configure forwarding destinations. It specifies where events should be sent.

-   **Indexers**: The destination indexers.
-   **Groups**: Defines groups of indexers.
-   **Routing Rules**: Specifies which events should go to which groups.

**Example:**
```ini
defaultGroup = my_indexer_group

[tcpout:my_indexer_group]
server = indexer1:9997, indexer2:9997 

# Below we are sending ONLY logs that match the REGEX defined in the transforms.conf file
[tcpout:target_group]
server = indexer3:9997
```
### Example Configuration
If you needed to route and filter logs based on a keystone/domain ID from OpenStack you could use this config to have indexer02 receive ONLY the logs that contain that ID and indexer01 to receive all other logs PLUS a clone of the logs sent to indexer02.

#### props.conf
```ini
[openstacklogs]
TRANSFORMS-routing = set_routing_tenant_id
```
#### transforms.conf
```ini
[set_routing_tenant_id]
REGEX = .(<domain_ID>)
FORMAT = tenant_id_group, ime-indexers
DEST_KEY = _TCP_ROUTING
```
#### outputs.conf
```ini
defaultGroup = ime-indexers
indexAndForward = true
autoLBFrequency = 30
clientCert = /path
sslCertPath = /path
sslRootCAPath = /path

# Becuase ime-indexers is the "defaultGroup" it will get ALL logs but notice in transforms.conf we also had to specify the "ime-indexers" group in the "set_routing_tenant_id" block. If not there the deaultGroup will NOT recieve the filtered logs getting sent to indexer02.

[tcpout:ime-indexers]
server = indexer01:9997
sslCertPath = /path
sslRootCAPath = /path

# Here we are sending ONLY logs that match the REGEX defined in the transforms.conf file.

[tcpout:tenant_id_group]
server = indexer02:9997
disabled = false
sslCertPath = /path
sslRootCAPath = /path
```
## Additional Resources

- [Splunk Routing and Filtering Docs](https://docs.splunk.com/Documentation/Splunk/9.2.1/Forwarding/Routeandfilterdatad)
