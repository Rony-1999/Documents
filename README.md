# Build o1-adapter

*In this method we are building o1-adapter bare-metal*

Assuming that all the libraries are already installed into the system, building the source code requires running the `./build.sh` script from within the `src` folder.
The script requires and accepts *no* parameters, and will compile and build the entire code.

Required libraries:
- pthread
- libyang
- sysrepo
- curl
- telnet
- cjson

Some of the libraries already have their own dependencies, such as:
- libssh version 0.9.2
- libnetconf2
```
# build the binary
cd /o1-adapter/src
./build
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
List of DU and CU yangs to be installed in sysrepo are:
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
sysrepoctl -I
# List out yangs in sysrepo
sysrepoctl -l
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
- Directly Through ODLUX web application
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

*Using the above instructions we can configure the DU & CU parameters with the help of netopeer2-cli  which acts as a NETCONF client and updated changes can be seen over O1-Adapter*


  
  


