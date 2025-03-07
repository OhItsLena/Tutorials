---
title: Create a Resource Group and Connect AWS S3 Object Store to SAP AI (Client SDK)
description: Learn creation of resource group in SAP AI Core to enable multi-tenancy through SAP AI API Client SDK. Store datasets to AWS S3 and connect to SAP AI Core through SAP AI API Client SDK.
auto_validation: true
time: 15
tags: [ tutorial>license, tutorial>advanced, topic>artificial-intelligence, topic>machine-learning, software-product>sap-business-technology-platform ]
primary_tag: topic>artificial-intelligence
author_name: Dhrubajyoti Paul
author_profile: https://github.com/dhrubpaul
---

## Details
### You will learn
- How to create Resource group using SAP AI API Client SDK
- How to upload data to AWS S3 bucket
- How to connect AWS S3 to SAP AI Core with Object Store Secret

---

[ACCORDION-BEGIN [Step 1: ](Create resource group)]

Resource Groups represent a virtual collection of related resources within the scope of one SAP AI Core tenant.


Create resource group with name `tutorial`, execute the following python code on your Jupyter notebook cell

```PYTHON[4]
ai_api_client.rest_client.post(
    path="/admin/resourceGroups",
    body={
        "resourceGroupId": "tutorial" # Name of your resource group
    }
)
```

Example Output

```PYTHON
{
    'resource_group_id': 'tutorial',
     'tenant_id': '123fasdf-aaaa-bbbb-cccc-1234asdf',
     'zone_id': ''
}

```

> **IMPORTANT:** The `create resource group` request results in `Response: 202`, which means the backend server will take time(~30 sec) to create the group. List the resource group *(see below)* to see the status of creation

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 2: ](List existing resource groups)]

Execute the following python code on your Jupyter notebook cell

```PYTHON
ai_api_client.rest_client.get(
    path=f"/admin/resourceGroups"
)
```

Example Output

```PYTHON
{'count': 1,
 'resources': [{
    'resource_group_id': 'tutorial',
    'status': 'PROVISIONED',
    'status_message': 'All onboarding steps are completed.',
    'tenant_id': '123fasdf-aaaa-bbbb-cccc-1234asdf',
    'zone_id': ''
  ]}
}
```

[VALIDATE_1]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 3: ](Create AWS S3 Object Store using S3 Browser)]

AWS S3 Object Store is used to store data. Here will store dataset for training ML Models.

There are two ways you can create AWS S3 Bucket.

1. Through SAP BTP Cockpit.

2. Through AWS. Refer [AWS User Guide to S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)


Install S3 Browser to manage AWS S3 from your computer. [Download here](https://s3browser.com/)

Open S3 Browser and enter your credentials.  

!![enter s3 credentials in s3 bucket](img/s3/init.png)

If you don't have existing bucket create one and skip to next step.

!![s3 bucket info](img/s3/bucket-1.png)  

!![s3 bucket info2](img/s3/bucket-2.png)  

[DONE]
[ACCORDION-END]


[ACCORDION-BEGIN [Step 4: ](Upload training dataset to the S3 Object Store)]

1. Open S3 Browser.

2. Click **New Folder**. Create a folder named `tutorial`.

    !![path prefix](img/s3/path-prefix.png)

3. Create another folder name `data` inside the `tutorial`.

    !![data folder](img/s3/data.png).

4. Click on **Upload** > **Upload Files**. Upload `travel.csv` *(download from below)* inside `tutorial/data/`.

    Download Files

    | File Name | Link |
    | --- | --- |
    | `travel.csv` | [Download Here](https://raw.githubusercontent.com/SAPDocuments/Tutorials/master/tutorials/ai-core-aiapi-clientsdk-resources/travel.csv)

    !![upload](img/s3/data-2.png)

Final look of your AWS S3 bucket.  

!![final s3 look](img/s3/final.png)

[DONE]
[ACCORDION-END]

[ACCORDION-BEGIN [Step 5: ](Register AWS S3 Object Store to SAP AI Core resource group)]

Object Store Secret links your S3 Object Store with your resource group, hence ensure you have the resource group created before proceeding.

Get service key file for your AWS S3 bucket. The file will have contents similar to the snippet below.

> In case you are use SAP BTP to create AWS S3 object store, generate you service key for same from`BTP cockpit > BTP subaccount > Instances and Subscriptions > Instances > Credentials `.

```JSON
{
  "access_key_id": "ASDFASDFASDFASDF",
  "bucket": "asd-11111111-2222-3333-4444-55555555555",
  "secret_access_key": "asdfASDFqwerQWERasdfQWER",
  "host": "s3.amazonaws.com",
  "region": "us-east-1",
  "uri": "s3://ASDFASDFASDFASDF:asdfASDFqwerQWERasdfQWER@s3.amazonaws.com/asd-11111111-2222-3333-4444-55555555555",
  "username": "asd-s3-123456-78910-aaa3-dddd-asdf12345"
}
```

Save the file locally as. `s3_service_key.json` inside the `files` folder: `files/s3_service_key.json`

!![service key](img/s3/s3-service-key.png)

Execute the following python code on your Jupyter notebook cell

```PYTHON
# Loads your service key
s3_service_key_path = 'files/s3_service_key.json'

# Loads the service key file
with open(s3_service_key_path) as s3sk:
    s3_service_key = json.load(s3sk)


default_secret = {
    "name": "default", # Name of the connection
    "type": "S3",
    "endpoint": s3_service_key["host"],
    "bucket": s3_service_key["bucket"],
    "pathPrefix": "tutorial",
    "region": s3_service_key["region"],
    "data": {
        "AWS_ACCESS_KEY_ID": s3_service_key["access_key_id"],
        "AWS_SECRET_ACCESS_KEY": s3_service_key["secret_access_key"]
    }
}

# Call the api
ai_api_client.rest_client.post(
    path="/admin/objectStoreSecrets",
    body = default_secret, # defined above
    resource_group = "tutorial"
)

```

Example Output

```PYTHON
{'message': 'secret has been created'}
```

This will connect your object store. The connection will be named `default`.

[DONE]
[ACCORDION-END]

---
