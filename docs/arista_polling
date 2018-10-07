The following is an example of polling an Arista vEOS switch.
This assumes panoptes is already setup.

Arista vEOS 4.20.7M was used for testing

#### To Do:

 -  
 

##### vEOS

vEOS management was already setup, and the following snmp config added:

```
snmp-server chassis-id veos2
snmp-server contact lab
snmp-server location lab
snmp-server community public ro
```

Confirm that snmp is working on vEOS switch:

```
snmpwalk -v 2c -c public -O e <veos-ip> SNMPv2-MIB::sysDescr.0
```

Should get back

```
SNMPv2-MIB::sysDescr.0 = STRING: Arista Networks EOS version 4.20.7M running on an Arista Networks vEOS
```


##### Add device to discovery file

```bash
nano /home/panoptes/plugins/discovery/localhost.json
```

The contents should be like so, where localhost was already added and the new switch is the 2nd entry:

```bash
[
  {
    "resource_plugin": "plugin_discovery_from_json_file",
    "resource_site": "local",
    "resource_class": "system",
    "resource_subclass": "host",
    "resource_type": "generic",
    "resource_id": "localhost",
    "resource_endpoint": "localhost",
    "resource_creation_timestamp": "1512629517.03121",
    "resource_metadata": {
      "_resource_ttl": "900"
    }
  },
  {
    "resource_plugin": "plugin_discovery_from_json_file",
    "resource_site": "local",
    "resource_class": "network",
    "resource_subclass": "switch",
    "resource_type": "arista",
    "resource_id": "veos",
    "resource_endpoint": "192.168.0.199",
    "resource_creation_timestamp": "1512629517.03121",
    "resource_metadata": {
      "_resource_ttl": "900"
    }
  }
]
```

##### Add Arista interface enrichment plugin

```bash
mkdir -p plugins/enrichment/interface/arista
touch plugins/enrichment/interface/arista/__init__.py
nano plugins/enrichment/interface/arista/plugin_enrichment_interface_arista.panoptes-plugin.example
```

Add the following to the file:

```
[Core]
Name = Arista Interface Enrichment Plugin
Module = plugin_enrichment_interface_arista

[Documentation]
Author = Oath, Inc.
Version = 0.1
Website = https://github.com/yahoo/panoptes
Description = Plugin to collect interface enrichment for arista devices

[main]
execute_frequency = 300
resource_filter = resource_class = "network" AND resource_type = "arista"
enrichment_ttl = 900

[snmp]
max_repetitions = 25
timeout = 10
retries = 2
```

```bash
nano plugins/enrichment/interface/arista/plugin_enrichment_interface_arista.py
```

Add the following to the file:

```bash
from .....plugins.enrichment.interface.plugin_enrichment_interface import PluginEnrichmentInterface


class PluginEnrichmentAristaInterface(PluginEnrichmentInterface):
    """
    InterfaceEnrichment class for Arista devices.
    """
    def get_parent_interface_name(self, index):
        """
        Gets the parent interface name for the interface associated with the provided index

        Args:
            index (int): The index used to look up the associated interface in self._interface_table

        Returns:
            string: The name of the parent interface, or self._MISSING_VALUE_STRING if the interface has no parent.
                    For Arista devices, this is everything to the left of the '/' in the interface name, if a '/' is
                    present.
        """
        interface_name = self.get_interface_name(index)
        if '/' in interface_name:
            return interface_name.split('/')[0]
        return self._MISSING_VALUE_STRING

    def get_parent_interface_configured_speed(self, index):
        parent_name = self.get_parent_interface_name(index)
        if parent_name is not self._MISSING_VALUE_STRING:
            return 4 * self.get_configured_speed(index)
        return self._MISSING_METRIC_VALUE

    def get_parent_interface_media_type(self, index):
        return self.get_media_type(index)

    def get_parent_interface_port_speed(self, index):
        return self.get_parent_interface_configured_speed(index)
```
