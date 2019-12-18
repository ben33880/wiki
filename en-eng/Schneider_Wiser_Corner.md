# Schneider WISER system

## Purpose

Purpose of this page is to share all findings regarding the WISER Zigbee product familly.

* [Thermostat](https://www.se.com/fr/fr/product/EER51000/wiser---thermostat/)
* [Smart Plug](https://download.schneider-electric.com/files?p_Doc_Ref=Wiser-Smartplug_EAV89774-01_EER400xx_10-2016_DA-EN-FR-IT-SV)

Work in progress, so only validated informations are written


## Pairing

* Channel: 15, 11
* Extended PANID: 0x6734484504015e10 
  Did test some alternate EPID without success!
  
There is also a strange thing which is the IEEE has the same prefix as the LIVOLO switches !

## Wiser Smart Plug

### Registration steps

1. Get Active Endpoint List
1. Simple Descriptor Request
1. Get Model name via read attribute 0x0000/0x0005
1. Bind 0x0019
1. Bind 0x0000
1. Bind 0x0009
1. Bind 0x0003
1. Bind 0x0006
1. Bind 0x0702
1. Configure Reporting on Cluster 0x0702 Attribute 0x0000 ( Min: 600, Max: 600)
1. Configure Reporting on Cluster 0x0702 Attribute 0x0400 ( Min:  30, Max: 600)
1. Configure Reporting on Cluster 0x0006 Attribute 0x0000 ( Min: 0, Max 600)
1. Read Attribute 0x0000 Attribute 0x0007 (Power Source)
1. Read Attribute 0x0000 Attribute 0x0013 (Alarm Mask)
1. Read Attribute 0x0000 Attribute 0xe000 (Version logicielle processeur réseau)
1. Read Attribute 0x0000 Attribute 0xe001 (Version logicielle application)
1. Read Attribute 0x0000 Attribute 0xe002 (Version matérielle application)
1. Read Attribute 0x0702 Attribute 0x0000, 0x0400, 0x0300, 0x0301, 0x0302
1. Read Attribute 0x0702 Attribute 0x5a20 (Schneider Specific)
1. Read Attribute 0x0006 Attribute 0x0000 (ON/OFF)
1. Write Attribute 0x0000 Attribute 0xe050 ( Data Type: Bool 0x10; Value: True 0x01 )
1. Write Attribute 0x0000 Attribute 0x0010 (Location Description) ( Data Type: String 0x42; Value: string )


## Wiser Thermostat

### Registration process

The HUB during the pairing process seems to be doing a number of actions on the End Device, something like registration

1. Write Attribute #1

1. Write Attribute #2


### Device Informations

```
ProfileID : "0104"
ZDeviceID : "0302"
Manufacturer : "105e"
MacCapa: 0x80
DeviceType : "RFD"
LogicalType : "End Device"
PowerSource : "Battery"
ReceiveOnIdle : "Off"
Stack Version : "2"
ZCL Version : "1"
Max Buffer Size : "50"
Max Rx : "00a0"
Max Tx : "00a0"
Manufacturer Name : "Schneider Electric"
Model: EH-ZB-RTS

Ep: 0b
Cluster IN Count: 07
Cluster In 1: 0000 (Basic)
Cluster In 2: 0001 (Power Configuration)
Cluster In 3: 0003 (Identify)
Cluster In 4: 0009 (Alarms)
Cluster In 5: 0204 (Thermostat User Interface Configuration)
Cluster In 6: 0402 (Temperature Measurement)
Cluster In 7: fe02
Cluster OUT Count: 02
Cluster Out 1: 0019 (Over-the-Air Upgrade)
Cluster Out 2: 0201 (Thermostat)
```

### Decoded scenario ( Schneider HUB, 1 Thermostat, 2 Actuators )

1. Thermostat sending local Temperature to:
   1. Hub by Report Attribute Cluster 0x0402 Attribute 0x0000
   1. actuators by Report Attribute Cluster 0x0402 Attribute 0x0000
   
1. Thermostat requesting Attribute from the Hub
   1. OccupedHeatingSetPoint by Read Attribute Cluster 0x0201 on Attribute 0x0012
   1. 0xe010 by Read Attribute Cluster 0x0201 on Attribute 0xe010 ( Hub returning Getting 8-Bit enum value: 0x01 )
   1. MaxHeatSetpointLimit by Read Attribute Cluster 0x0201 on Attribute 0x0016 ( Hub returning response 3500 )
   1. MinHeatSetpointLimit by Read Attribute Cluster 0x0201 on Attribute 0x0015 ( Hub returning response 50 ) 

1. each Actuators sending Instantaneous Demand to:
   1. Hub by Report Attribute Cluster 0x0702 Attribute 0x0400
   
At that stage, someone set a target temperature on the Application managing the HUB

1. Thermostat sending SetPoint to:
   1. HUB by using the command 0xe0 on Cluster 0x0201
   1. Actuators by using the command 0xe0 on Cluster 0x0201
   
1. HUB sending SetPoint to:
   1. Actuators by using the command 0xe0 on Cluster 0x0201
   
1. each Actuators sending Instantaneous Demand to
   1. Hub by Report Attribute Cluster 0x0702 Attribute 0x0400
   
#### comments

* The HUB is acting as a Zigbee controler but not only as it is responding to cluster 0x0201 (Thermostat)
* Can the Zigate implement Cluster 0x0201 and store Attributes 0x0012, 0x0015, 0x0016 as well as when receiving command 0xe0 store the value into attribute 0x0012
* We should investigate what are the various values of Cluster 0x0201 on Attribute 0xe010 ( 8bit enum)



## Decoding specifc cluster

### Decoding Cluster 0x0201 on Attribute 0xe010

* Data Type: 0x30 (8bit enum)
* 0x06 - Vacation
* 0x05 - 
* 0x04 - 
* 0x03 - Economie
* 0x02 - Programme
* 0x01 - Manuel
* 0x00 - Normal

### Decoding Cluster 0x0201 Command Specific 0xe0

* This command seems to be used to set Setpoint 
  ```
  ZigBee Cluster Lib Frame: 0x11 
  Sequence number         : 0xc1
  Command                 : 0xe0 
                          : 0x00
  Length                  : 0x01
  Value                   : 0x34 0x08 (Setpoint with LBHB , don't forgett the Endian ;-)
                          : 0xff
  ```   
  
  
  Sources of information:
  * https://studylibfr.com/doc/2872316/compteur-d-%C3%A9nergie-sans-fil-s%C3%A9rie-em4300
                        