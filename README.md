# Setup o1-adapter

**o1-adapter has been built using the bare-metal method**

Assuming that all the libraries are already installed into the system, building the source code requires running the `./build.sh` script from within the `src` folder.
The script requires and accepts *no* parameters, and will compile and build the entire code.

Required libraries:
- pthread
- libyang
- sysrepo
- curl
- telnet
- cjson

Some of the libraries already have their dependencies, such as:
- libssh version 0.9.2
- libnetconf2
```
# build the binary
cd /o1-adapter/src
./build
```
### How to connect o1-adapter and OAI via Telnet
Specialized functionality for O1 NETCONF Server is provided by the OAI Telnet (acts as telnet server) extension for O1. The attributes are prepared and converted to the format and structure required by the relevant yang specification.

OAI telneto1: https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/telnet-o1/common/utils/telnetsrv/DOC/telneto1.md
```
# Bring up o1-adapter(DU)
cd /o1-adapter/src
./gnb-adapter --du

# Bring up o1-adapter(CU)
cd /o1-adapter/src
./gnb-adapter --cu
```

### Building and installing dependencies
For building the binary, such examples include `libpcre2-dev`, `zlib1g-dev`, and `libssl-dev`.

Some tools are required for running the binary with its NETCONF dependencies, such as `openssl`, `openssh-client`, `openssh-server`, and `vsftpd`.

After all the dependencies are met, running the `netconf_dep_install.sh` script will automatically build and install the following libraries (versions can vary upon update, but MAJOR must match):
- libssh v0.9.2
- libyang v2.1.30
- sysrepo v2.2.36
- libnetconf2 v2.1.28
- netopeer2 v2.1.49
- curl v7.87.0
- cJSON v1.7.16
- libtelnet v0.23

After building and installing dependencies, the YANG models must be retrieved and installed.

### Retrieving and installing the O1 YANG Models
Retrieving the O1 YANG models from 3GPP can be done using the `get-yangs.sh` script.

Once the YANG models are available the `install-yangs.sh` script will install them.

Installing the models must be done via such scripts because the script only installs the required models, not all of them, and the install order is important.

### Installing DU & CU 3GPP YANGS in Sysrepo
A list of DU and CU yangs to be installed in sysrepo are:
|YANGS |
| --- |
_3gpp-common-yang-extensions.yang
_3gpp-common-yang-types.yang
_3gpp-common-files.yang
_3gpp-common-measurements.yang
_3gpp-common-subscription-control.yang
_3gpp-common-fm.yang
_3gpp-common-trace.yang
_3gpp-5gc-nrm-configurable5qiset.yang
_3gpp-5gc-nrm-ecmconnectioninfo.yang
_3gpp-common-managed-element.yang
_3gpp-common-managed-function.yang
_3gpp-nr-nrm-gnbdufunction.yang
_3gpp-nr-nrm-gnbcucpfunction.yang
_3gpp-5g-common-yang-types.yang
_3gpp-nr-nrm-nrcelldu.yang
_3gpp-nr-nrm-nrcellcu.yang
_3gpp-nr-nrm-bwp.yang
_3gpp-nr-nrm-nrsectorcarrier.yang
_3gpp-nr-nrm-nrcellrelation.yang
_3gpp-nr-nrm-nrfreqrelation.yang
_3gpp-common-ep-rp.yang
_3gpp-nr-nrm-gnbcuupfunction.yang
_3gpp-nr-nrm-ep.yang

```
# Installing yang in sysrepo
sysrepoctl -i <yang file>
# List out yangs in sysrepo
sysrepoctl -l <yang file>
```
### Configuration Management of DU & CU 
The O1-Adapter enables the SMO to perform essential O&M activities on O-DU and O-CU.

**Configuration Management:** Adding, modifying, or deleting configurations of connected O-DU and O-CU.

List of DU and CU configurable parameters:

DU Parameters

|Class |Attributes|
| -- | -- |
GNBDUFunction	|gNB­DUId
GNBDUFunction	|gNBDUName
GNBDUFunction	|gNBId
GNBDUFunction	|gNBIdLength 
NRCellDU	|cellLocalId
NRCellDU	|pLMNIdList
NRCellDU	|nRPCI
NRCellDU	|nRTAC
NRCellDU	|arfcnDL
NRCellDU	|arfcnUL
NRCellDU	|arfcnSUL
NRCellDU	|bSChannelBwDL 
NRCellDU	|ssbFrequency
NRCellDU	|ssbPeriodicity
NRCellDU	|ssbSubCarrierSpacing
NRCellDU	|ssbOffset
NRCellDU	|bSChannelBwUL
NRCellDU	|bSChannelBwSUL
NRSectorCarrier	|arfcnDL
NRSectorCarrier	|arfcnUL
NRSectorCarrier	|bSChannelBwDL
NRSectorCarrier	|bSChannelBwUL
BWP	|isInitialBwp
BWP	|subCarrierSpacing
BWP	|cyclicPrefix
BWP	|startRB
BWP	|numberOfRBs
NRFrequency	|absoluteFrequencySSB
NRFrequency	|sSBSubCarrierSpacing

CU Parameters

|Class |Attributes|
| -- | -- |
GNBCUCPFunction	|gNBId
GNBCUCPFunction	|gNBIdLength 
GNBCUCPFunction	|gNBCUName
GNBCUCPFunction	|pLMNId
GNBCUUPFunction	|gNB­CUUPId
GNBCUUPFunction	|pLMNIdList
GNBCUUPFunction	|gNBId
GNBCUUPFunction	|gNBIdLength 
NRCellCU	|cellLocalId
NRCellCU	|pLMNIdList

### Running the adapter and the NETCONF server
As the `adapter-entrypoint.sh` script illustrates, two daemons must be run to allow an O1 NETCONF Server:
- netopeer2-server, which is the NETCONF server itself, with all the models installed
- The default netconf server timeout is 5 seconds, which is not enough for long requests, that's why the netopeer2-server must have `-t 60` parameter which changes the timeout to 60 seconds.
  ```
  # start netconf server
  netopeer2-server -d -v3 -t 60
  ```
- gnb-adapter, which is the adapter that connects to gNB via telnet and handles the NETCONF server's requests for data

### How to connect via O1

Using any NETCONF client, one should connect to the adapter by connecting to the corresponding host/IP address and port.

There are two ways to configure the parameters:

- Using netopeer2-cli:
```
  # command to start cli
    sudo netopeer2-cli 
  # login to cli
    connect --host <ip-address> --login netconf
  # get the existing config 
    get-config --source running <--filter-xpath /_3gpp-common-managed-element:ManagedElement/*>
  # edit the configuration
    edit-config --target running --config --defop replace.
 ```
- Directly Through the ODLUX web application
```
* Connect the SMO GUI with the host IP at port 8080 and check the connection of DU & CU in connect
* If CU & DU are connected then go to Configuration and check for the 3gpp_ yangs and try to configure them.
* Check for the updated changes over the O1-adapter
```
*Using the above instructions we can configure the DU & CU parameters with the help of netopeer2-cli  which acts as a NETCONF client and updated changes can be seen over O1-Adapter*

**Calculated Configurations acording to Bandwidths: https://drive.google.com/file/d/1RUGTTSzs_5HsXVsqmXQj2USwjv_uQF9U/view?usp=sharing**

### Performance management of DU
Performance management -describes measurements and counters used to collect data related to DU/CU operations

Data Flow: https://docs.o-ran-sc.org/projects/o-ran-sc-nonrtric-plt-ranpm/en/latest/overview.html#data-flow

When any UE connects with the network its operational information is collected in the form of a file-ready format in the PM data collector of SMO.

  * PM is collected
     * in files, provided via FTP
     * Notification via VES if a new file is available.
     * Notifications are running always after the O1Adapter startup and the Order of notifications is as follows:
       
        *PNF Registration, Heartbeat Event, File Ready Event, Fault Management*

 **Mapping of PM data** 
 |Name |JSON Type |Description    |
 | -- | -- | --     |
 frame-type 	|enum ["tdd","fdd"] 	|not used
band-number 	|number 	
num-ues       |number 	|Number of connected UEs (To cell or DU?). See the section "Performance Data (PM)"
ues 	 	|List |	
load 	 	|integer 	|Percentage number [0..100], reflecting downling load
ues-thp.rnti |List 	|List with throughput status of UEs. Virtual class: setup specified below.
uethp:rnti 	 |Integer 	|UE is
uethp:dl 	|Integer 	|Downlink load in kbit/s
uethp:ul 	|Integer 	|Uplink load kbit/s



### VES Notifications
A virtual Event Streaming (VES) Collector is a RESTful collector that processes JSON messages/VES messages.

Reference: https://docs.o-ran-sc.org/projects/o-ran-sc-nonrtric-plt-ranpm/en/latest/overview.html#data-flow

  **Explanation of VES messages and VES collector request and response:**
 - Run OAI telnet server and O1-Adapter telnet client
 - Run netopeer server
 - Run SMO
 - Posting VES events such as PNF registration, heartbeat, and file ready to SMO with Response = 202 and with message successfully event sent

![image](https://github.com/user-attachments/assets/b6bcd363-ddef-4c97-a88f-9fff1dfea7e5)

### Configuration
Configuration is done through the `config.json` file. A sample is provided:
```
{
    "log-level": 3,
    "software-version": "abcd",
    "network": {
        "host": "10.20.11.134",
        "username": "netconf",
        "password": "netconf!",
        "netconf-port": 1830,
        "sftp-port": 1222
    },

    "ves": {
        "template": {
            "new-alarm": "/adapter/config/ves-new-alarm.json",
            "clear-alarm": "/adapter/config/ves-clear-alarm.json",
            "pnf-registration": "/adapter/config/ves-pnf-registration.json",
            "file-ready": "/adapter/config/ves-file-ready.json",
            "heartbeat": "/adapter/config/ves-heartbeat.json",
            "pm-data": "/adapter/config/pmData-measData.xml"
        },

        "pnf-registration": true,
        "heartbeat-interval": 30,

        "url": "https://10.20.35.128:8443/eventListener/v7",
        "username": "sample1",
        "password": "sample1",

        "file-expiry": 86400,
        "pm-data-interval": 30
    },

    "alarms": {
        "internal-connection-lost-timeout": 3,

        "load-downlink-exceeded-warning-threshold": 50,
        "load-downlink-exceeded-warning-timeout": 30
    },

    "telnet": {
        "host": "10.20.11.134",
        "port": 9091
    },

    "info": {
        "gnb-du-id": 0,
        "gnb-cu-id": 0,
        "cell-local-id": 0,
        "node-id": "gNB-Eurecom-5GNRBox-00001",
         "node-id": "gNB-Eurecom-5GNRBox-00002",
        "location-name": "MountPoint 05, Rack 234-17, Room 234, 2nd Floor, Körnerstraße 7, 10785 Berlin, Germany, Europe, Earth, Solar-System, Universe",
        "managed-by": "ManagementSystem=O-RAN-SC-ONAP-based-SMO",
        "managed-element-type": "NodeB",
        "model": "nr-softmodem",
        "unit-type": "gNB"
        "cu-unit-type": "CU"
    }
}
```

**Configurations in CU and DUs config.json to look for before running CU and DU adapter**
  - url needs to be modified according to requirements as the webpage is secured or not
  - username and password can be modified as per user's choice
  - while running CU and DU simultaneously telnet port should be different for both
  - If working on fault management modification in alarm load threshold and timeout needs to be done as per requirement
    ```
    # for CU
    "telnet": {
        "host": "10.33.42.242",
        "port": 9090
    },
    #for DU
    "telnet": {
        "host": "10.33.42.242",
        "port": 9091
    },
    ```
## Appendix

### References

