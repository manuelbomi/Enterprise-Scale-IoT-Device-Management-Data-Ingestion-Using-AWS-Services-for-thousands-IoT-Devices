# Enterprise-Scale IoT Device Management & Data Ingestion Using AWS Services for 10K+ Devices (Sensors, RFID, Raspberry Pis) 

#### This repository contains a complete, repeatable structure and scripts to automatically provision, onboard, simulate, and ingest data from a heterogeneous fleet of IoT devices. 

#### To start with, we are considering about 200 heterogenous IoT devices that are grouped into logical groups. Without loss of generality, our approach here is scalable to tens of thousands IoT/IIoT devices.  

#### It uses AWS IoT Core, AWS IoT Device Management (Fleet Provisioning), IoT Rules, and AWS storage services (S3) to route and persist telemetry. 

#### The repo also includes scripts (Python + AWS IoT Device SDK v2) and templates to organize/group devices and validate the pipeline.

### High-Level Objectives

#### The high level objectives include:
    • 1. Automatically onboard 200+ devices (industrial sensors, RFID readers, Raspberry Pis, etc.) grouped into 6 categories
    • 2. Publish sensor data to AWS IoT Core, using MQTT topics per device/group
    • 3. Ingest, transform, and store data (e.g., in S3) for downstream processing

## Step-by-Step Guide for Full Automation of Onboarding the Devices to AWS IoT Core
#### Requirements
Description
✅ AWS IoT Core & IoT Device Management Enabled
Use AWS Console/CLI
✅ AWS IoT Fleet Provisioning Enabled
For bulk onboarding
✅ IoT Policy Created
For MQTT access
✅ S3 Bucket Created
To store device data
✅ AWS CLI + Python SDK Installed
On your provisioning computer or edge gateway

| Header 1 | Header 2 | Header 3 |
|---|---|---|
| Row 1, Col 1 | Row 1, Col 2 | Row 1, Col 3 |
| Row 2, Col 1 | Row 2, Col 2 | Row 2, Col 3 |

   
