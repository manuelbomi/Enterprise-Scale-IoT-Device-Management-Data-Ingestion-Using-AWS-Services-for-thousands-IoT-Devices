# Enterprise-Scale IoT Device Management & Data Ingestion Using AWS Services for 10K+ Devices (Sensors, RFID, Raspberry Pis) 

#### This repository contains a complete, repeatable structure and scripts to automatically provision, onboard, simulate, and ingest data from a heterogeneous fleet of IoT devices. 

#### To start with, we are considering about 200 heterogenous IoT devices that are grouped into logical groups. Without loss of generality, our approach here is scalable to tens of thousands IoT/IIoT devices.  

#### It uses AWS IoT Core, AWS IoT Device Management (Fleet Provisioning), IoT Rules, and AWS storage services (S3) to route and persist telemetry. 

#### The repo also includes scripts (Python + AWS IoT Device SDK v2) and templates to organize/group devices and validate the pipeline.

### <ins>High-Level Objectives</ins>

#### The high level objectives include:
    • 1. Automatically onboard 200+ devices (industrial sensors, RFID readers, Raspberry Pis, etc.) grouped into 6 categories
    • 2. Publish sensor data to AWS IoT Core, using MQTT topics per device/group
    • 3. Ingest, transform, and store data (e.g., in S3) for downstream processing

## Step-by-Step Guide for Full Automation of Onboarding the Devices to AWS IoT Core
#### Requirements / Prerequisites


| Requirements | Description| 
|---|---|
| AWS IoT Core & IoT Device Management | Use AWS Console/CLI | 
| AWS IoT Fleet Provisioning Enabled | For bulk onboarding |
|IoT Policy Created | For MQTT access | 
| S3 Bucket Created | To store device data |
|AWS CLI + Python SDK Installed |On your provisioning computer or edge gateway |

### STEP 1: Define Your Device Fleet and Groups
For example:

| Group | Type | Count | Description| 
|---|---|---|---|
| group1 | Temperature Monitors | 40 |Sensor nodes in cold storage| 
| group2 |Industrial Vibration Sensors |30 | Motors & heavy machinery| 
|group3 | RFID Tags/Readers | 30 | Asset tracking| 
| group4 | Raspberry Pi Nodes |40 | Edge compute and cameras | 
|group5 |Smart Meters / Power Sensors |30 | Grid and power usage | 
|group6 |Environmental Sensors |30 | Humidity, CO2, pressure | 

#### Each group will have its own:
    • Thing Group
    • Fleet Provisioning Template
    • Topic structure for MQTT (iot/group1/device_id/data)
    • IoT Policy

### STEP 2: Create a Global IoT Policy
Create a policy GenericIoTPolicy allowing MQTT access:
---
```ruby
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "iot:Connect",
      "iot:Publish",
      "iot:Subscribe",
      "iot:Receive",
      "iot:GetThingShadow",
      "iot:UpdateThingShadow"
    ],
    "Resource": "*"
  }]
}


aws iot create-policy \
  --policy-name GenericIoTPolicy \
  --policy-document file://iot_policy.json

```
---






  
  

   
