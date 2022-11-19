## **CTRL + C on terminal**

![](https://i.ibb.co/6F2JRF0/Ctrl-C-1-final.png)

### **Step 1: SIGTERM signal will trigger the stop process**

### **Step 2: Stop all extension of zigbee2mqtt**

-   Removing listener from listerner array of all events registered on each extension : Bridge, publish, receive, deviceGroupMembership, configure, networkMap, groups, bind, onEvent, OTAUpdate, report, externalExtension.

-   Controller: adapterDisconnect, lastSeenChanged
-   State: deviceLeave
-   MQTT: publishAvailability

### **Step 4: Stop state**

-   Remove listener of events registered in State (deviceLeave)
-   Clear interval of periodically saving state
-   Saving state to file

### **Step 5: Disconnect MQTT**

-   Publish “offline” to MQTT topic ‘bridge/state’
-   Remove listener for events registered of MQTT: publishAvailability

### **Step 6:**

-   Save database
-   Remove all listener of adapter events: deviceJoined, zclData, rawData, disconnected, deviceAnnounce, deviceLeave.
-   Disable device join
-   Clear interval of periodically backing up and saving database
-   Writing coordinator backup to file

### **Step 7: Emit event**

-   Call adapter.stop() to emit ‘close’ event. Do nothing after that.

### **Step 8: Stop process**


---

# **ADAPTER EVENTS**
![](https://i.ibb.co/WpgwMn0/adapter-events-drawio.png)
Adapter events include:
- [disconnected](#unplug-the-usb)
- [deviceJoined](#new-device-join)
- [deviceAnnounce](#device-announce)
- [networkAddress](#send-zcl-command-to-device-failed-because-network-address-changed-while-devices-still-on-zigbee-network)
- [deviceLeave](#device-leave)
- [zclData](#adapter-receive-af-message-from-device)
- [rawData](#adapter-receive-af-message-from-device)

#### While [disconnected](#unplug-the-usb) is triggered by [UNPLUG THE USB](#unplug-the-usb), all the rest of events are triggered when adapter receive message from devices
---

## **UNPLUG THE USB**

**_Adapter Event: adapterDisconnected_**
![](https://i.ibb.co/ssbqdhM/UNPLUG.png)

### **Step 1: Emit and listen to events**

### **Step 2: Stop controller**

-   Do the same as [Step 2](#step-2-stop-all-extension-of-zigbee2mqtt) -> [Step 6](#step-6) of [CTRL + C process](#ctrl--c-on-terminal)
-   Stop the process
---
## **New Device Join**

**_Adapter Event: deviceJoined_**

**_Trigger when adapter receive cmd_**: Type=AREQ, Subsystem=ZDO, command=tcDeviceInd
![](https://i.ibb.co/tQXwphQ/device-Joined.png)

### **Step 1: Listen to events and trigger callback function**
### **Step 2: Create device**
- Create device if ieeeAddr not exist in database
- undelete if deleted device joined
### **Step 3: Execute callback function for onDeviceJoined event of eventBus**

-   Publish Device's info to **bridge/devices** topic with payload:
    _ieee_address, type, network_address, supported, friendly_name, description, definition, power_source, software_build_id, date_code,model_id, interviewing, interview_completed, manufacturer, endpoints._

-   Publish Event's info to **bridge/event** topic with payload:
    _friendly_name, ieee_address, type._
-   configure device
-   run onEvent function of device

### **Step 4: Update nwkAddr in database**
### **Step 5: Update last seen**
### **Step 6: Send pending request for each endpoint**
### **Step 7: Interview device**

---

## **Device Announce**

**_Adapter Event: deviceAnnounce_**

**_Trigger when adapter receive cmd_**: Type=AREQ, Subsystem=ZDO, command=endDeviceAnnceInd
![](https://i.ibb.co/W0MD4bH/announce.png)
### **Step 0: Discover route then emit event**
### **Step 1: Listen to events and trigger callback function**
### **Step 2: Initialize device by ieeeAddr**
### **Step 3: Update last seen**
### **Step 4: Send pending request for each endpoint**
### **Step 5: Update nwkAddr in database**
### **Step 6:**
-   Publish Device's info to **bridge/devices** topic with payload:
    _ieee_address, type, network_address, supported, friendly_name, description, definition, power_source, software_build_id, date_code,model_id, interviewing, interview_completed, manufacturer, endpoints._

-   Publish Event's info to **bridge/event** topic with payload:
    _friendly_name, ieee_address, type._
-   run onEvent function of device

---

## **Send ZCL command to device failed because network address changed (while device's still on zigbee network)**

**_Adapter Event: networkAddress_**

**_Trigger when adapter receive cmd_**: Type=AREQ, Subsystem=ZDO, command=nwkAddrRsp

**_NwkAddr Change (while not leave zigbee network) happen when_**: 
- Network Address conflict
- Network Address is changed in database while GW is powered off. (When GW need nwkAddr first time after start up, _for e.g: to send zcl cmd to device_, it will take from database)


![](https://i.ibb.co/WfzYpcR/address-drawio.png)

### **Step 1: Listen to events and trigger callback function**
### **Step 2: Initialize device by ieeeAddr**
### **Step 3: Update nwkAddr in database**
### **Step 4:**
-   Publish Device's info to **bridge/devices** topic with payload:
    _ieee_address, type, network_address, supported, friendly_name, description, definition, power_source, software_build_id, date_code,model_id, interviewing, interview_completed, manufacturer, endpoints._

-   run onEvent function of device

---

## **Device Leave**

**_Adapter Event: deviceLeave_**

**_Trigger when adapter receive cmd_**: Type=AREQ, Subsystem=ZDO, command=leaveInd



![](https://i.ibb.co/hBNNz7g/leave-drawio.png)

### **Step 1: Listen to events and trigger callback function**

### **Step 2: Delete device from database**

### **Step 3:**
- Delete device by ieeeAddr in state array
- Publish Device's info to **bridge/devices** topic with payload:
    _ieee_address, type, network_address, supported, friendly_name, description, definition, power_source, software_build_id, date_code,model_id, interviewing, interview_completed, manufacturer, endpoints._

-   Publish Event's info to **bridge/event** topic with payload:
    _friendly_name, ieee_address, type._
-   Clear timeOut of device

---

## **Adapter receive AF message from device**
**_Adapter Event: zclOrRawData_**

**_Trigger when adapter receive cmd_**: Type=AREQ, Subsystem=AF, command=incomingMessage || incomingMessageExt
![](https://i.ibb.co/VmbXjsM/zcl.png)


### **Step 1: Listen to events and trigger callback function**

### **Step 2: Delete Cluster field of payload frame if datatype is ZCL**

### **Step 3: Special handle for cluster Touchlink or Green Power**
- If cluster is **Touchlink** => Return (it will be handled at another function)
- If cluster is **Green Power** => handle with different types of Green Power messages: GP Commissioning, GP Success, GP Channel Request, GP Manufacturer-specific Attribute Reporting

### **Step 4: Handling of re-transmitted Xiaomi messages**
- Some Xiaomi router devices re-transmit messages from Xiaomi end devices. The network address of these message is set to the one of the Xiaomi router.
- Therefore it looks like if the message came from the Xiaomi router, while in fact it came from the end device.
- Handling these message would result in false state updates. The group ID attribute of these message defines the network address of the end device.
### **Step 5: Update last seen**
### **Step 6: Send pending request of each endpoint**
### **Step 7: Create endpoint**
### **Step 8: Initialize & load data for variables**
- Variable frame and command are initialized & load data from dataPayload for further processing at later stages
### **Step 9.a: Skip duplicate IAS Zone message from PIR device**
- Added by CongNT16
### **Step 10.a: Get type and data of command**
- Get type and data of command from the frame info
- If frame is global, its types will be: read, write, report, readRsp
- If frame is specific, type of command should be looked up in **events.ts**
### **Step 11.a: Get attribute value of Basic cluster**
- Get these attributes value if they've not been gotten yet: modelID, manufacturerName, powerSource, zclVersion, applicationVersion, stackVersion, hardwareVersion, dateCode, softwareBuildID
### **Step 12.a: Save Cluster and Attribute value to CLuster List**
### **Step 13.a:**
_Do nothing if device type is Coordinator_
- Check available converter for Zigbee command
- Convert Zigbee message to MQTT
- Bound poll endpoints and group members for state changes.
- Run onEvent function of device
- Check if device support OTA update and any new OTA update available
### **Step 14.a: Emit lastSeenChangedEvent**
- reason: messageEmitted
### **Step 13.b:Save Cluster and Attribute value to CLuster List**
### **Step 14.b:Emit lastSeenChangedEvent**
- reason: messageNonEmitted
### **Step 15:**
- Update reportable properties
- Handle the respond to Enroll Request, Read Request
- Handle check-in from sleeping end devices
- Send a default response if necessary








