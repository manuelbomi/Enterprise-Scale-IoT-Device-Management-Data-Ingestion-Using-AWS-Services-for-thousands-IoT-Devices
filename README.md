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
Create a policy (e.g. GenericIoTPolicy) allowing MQTT access:

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


### STEP 3: Create Fleet Provisioning Template (1 per Group)

Create one provisioning template per device group (group1_template.json, etc.)

Example for group1:

---
```ruby
{
  "parameters": {
    "SerialNumber": {
      "type": "String"
    }
  },
  "resources": {
    "thing": {
      "type": "AWS::IoT::Thing",
      "properties": {
        "ThingName": {"Fn::Sub": "${SerialNumber}"}
      }
    },
    "certificate": {
      "type": "AWS::IoT::Certificate",
      "properties": {
        "Status": "ACTIVE"
      }
    },
    "policy": {
      "type": "AWS::IoT::Policy",
      "properties": {
        "PolicyName": "GenericIoTPolicy"
      }
    }
  }
}

```
---

Create the template for each group: 

```ruby
aws iot create-provisioning-template \
  --template-name group1_template \
  --template-body file://group1_template.json \
  --enabled

```
---

Repeat for group2_template, ..., group6_template. 

---

### STEP 4: Flash Devices with Python Provisioning Script + Claim Cert

Each device or gateway will be flashed with:

    • A claim certificate + private key + root CA
    
    • A Python provisioning script
    
    • A serial number or other unique ID

##### Sample provision_device.py (run on each device/gateway) 
---
```ruby

import json, time, uuid
from awscrt import io, mqtt
from awsiot import iotidentity, mqtt_connection_builder

# --- CONFIGURATION ---
TEMPLATE_NAME = "group1_template"   # Change per group
DEVICE_SERIAL = "temp-001"          # Unique ID
ENDPOINT = "xxxxxx-ats.iot.us-west-2.amazonaws.com"
CERT = "claim_cert.pem"
KEY = "claim_private_key.pem"
ROOT_CA = "AmazonRootCA1.pem"

mqtt_connection = mqtt_connection_builder.mtls_from_path(
    endpoint=ENDPOINT,
    cert_filepath=CERT,
    pri_key_filepath=KEY,
    ca_filepath=ROOT_CA,
    client_id=f"client-{DEVICE_SERIAL}",
    clean_session=True,
    keep_alive_secs=30
)

mqtt_connection.connect().result()
identity_client = iotidentity.IotIdentityClient(mqtt_connection)

# --- Register Device ---
certs = identity_client.create_keys_and_certificate().result()
register = identity_client.register_thing(
    template_name=TEMPLATE_NAME,
    parameters={"SerialNumber": DEVICE_SERIAL},
    certificate_ownership_token=certs.certificate_ownership_token
).result()

# --- Store New Certs ---
with open("device_cert.pem", "w") as f: f.write(certs.certificate_pem)
with open("device_key.pem", "w") as f: f.write(certs.private_key)
print(f"[✔] {DEVICE_SERIAL} registered as {register.thing_name}")

mqtt_connection.disconnect().result()

```
---



### STEP 5: MQTT Data Publishing (Device → AWS)

After provisioning, device can use saved certs to connect securely and publish MQTT messages:

---
```ruby

mqtt_connection = mqtt_connection_builder.mtls_from_path(
    endpoint=ENDPOINT,
    cert_filepath="device_cert.pem",
    pri_key_filepath="device_key.pem",
    ca_filepath=ROOT_CA,
    client_id=DEVICE_SERIAL,
    clean_session=False,
    keep_alive_secs=60
)

mqtt_connection.connect().result()

payload = {
    "device_id": DEVICE_SERIAL,
    "timestamp": int(time.time() * 1000),
    "sensor_data": {
        "temperature": 22.5,
        "humidity": 55.0
    }
}

mqtt_connection.publish(
    topic=f"iot/{DEVICE_GROUP}/{DEVICE_SERIAL}/data",
    payload=json.dumps(payload),
    qos=mqtt.QoS.AT_LEAST_ONCE
)


```
---

### STEP 6: Create AWS IoT Rules to Forward MQTT to S3 (i.e. subscription from MQTT to S3 Bucket)

Example SQL Rule:

```ruby
SELECT * FROM 'iot/group1/+/data'
```

Create rule: 

```ruby
aws iot create-topic-rule \
  --rule-name "group1_data_to_s3" \
  --topic-rule-payload '{
    "sql": "SELECT * FROM \"iot/group1/+/data\"",
    "ruleDisabled": false,
    "actions": [{
      "s3": {
        "roleArn": "arn:aws:iam::your-account-id:role/iot_s3_role",
        "bucketName": "your-iot-data-bucket",
        "key": "${topic()}/${timestamp()}.json"
      }
    }]
  }'

```

Ensure that:
    • You have an IAM role with iot.amazonaws.com as principal
    
    • Role has write access to the bucket

---

### STEP 7: Group Devices and Manage via IoT Device Management

Once devices are onboarded:

    • Add each Thing to a Thing Group:



  
  

   
