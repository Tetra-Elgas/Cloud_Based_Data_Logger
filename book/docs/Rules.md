# Rules

AllThingsTalk Platform comes with an integrated rules engine. A rules engine lets you automate work by defining actions that need to be taken in response to a set of triggers. For example, you could make it turn the lights on when you open the door, dispense cat food at predefined times, or notify you if your package is handled improperly in transport.

## Use case

![CO2 chart](.\img\CO2 chart.png)

[Reference](https://www.buildera.com/carbon-dioxide-co2-monitoring-service)

> *The CO2 content of the air should, ideally, not exceed **900 ppm** and under no circumstances should it exceed the standard of **1200 ppm**.*
>
> [reference](https://www.info-coronavirus.be/en/ventilation/)

The following rules are set:

![rules](.\img\Rules\rules.PNG)

References: 

- [get started with rules](https://www.allthingstalk.com/faq/get-started-with-rules) 
- [create rules to your needs](https://www.allthingstalk.com/faq/create-custom-rules-using-earl) 
- [create custom rules in EARL](https://www.allthingstalk.com/faq/create-custom-rules-using-earl)

## Cloud Setup

**Create Virtual asset**

First, create a virtual asset.

![AllThingsTalk_New_asset](.\img\Rules\AllThingsTalk_New_asset.PNG)



![AllThingsTalk_Create_Asset](.\img\Rules\AllThingsTalk_Create_Asset.PNG)



![AllThingsTalk_Assets](.\img\Rules\AllThingsTalk_Assets.PNG)

<br>

**Rules setup**

Click on Rules &rightarrow;  New Rule

![AllThingsTalk_Rules](.\img\Rules\AllThingsTalk_Rules.PNG)

Rule definition:

![AllThingsTalk_Rules_Def](.\img\Rules\AllThingsTalk_Rules_1.PNG)

![AllThingsTalk_Rules_Def](.\img\Rules\AllThingsTalk_Rules_2.PNG)

![AllThingsTalk_Rules_Def](.\img\Rules\AllThingsTalk_Rules_3.PNG)

![AllThingsTalk_Rules_Def](.\img\Rules\AllThingsTalk_Rules_3.PNG)

![AllThingsTalk_Rules_Def](.\img\Rules\AllThingsTalk_Rules_4.PNG)

On the Pinboard define a label: click on New Pin &rightarrow;  Label

![AllThingsTalk_New_Pin](.\img\Rules\AllThingsTalk_New_Pin.PNG)

Choose an asset = CO2_indicator

![AllThingsTalk_New_Pin_Asset](.\img\Rules\AllThingsTalk_New_Pin_Asset.PNG)

![AllThingsTalk_Pinboard](.\img\Rules\AllThingsTalk_Pinboard.PNG)

## Test

Below a Python test script to test the rule definition.

```Python
def decoder(num_values):
    value_cat = []
    for num_label in num_values:
        if num_label >= 1200:
            value_cat.append('Bad')
        elif num_label >= 900:
            value_cat.append('Moderate')
        else:
            value_cat.append('Good')
    return value_cat


def send_asset_data_to_cloud(device, asset_name, payload_timestamp, payload_value):
    """
    Send a value to an asset

    Parameters
    ----------
    device: type dict
        device description (defined in a json file)
    asset_name: str
        name of the assets
    payload_timestamp: str
        timestamp
    payload_value:
        payload value

    Returns
    -------
    message: str
        ConnectionOK: data is send successful
        ConnectionError: a Connection error occurred. (https://docs.python-requests.org/en/master/_modules/requests/exceptions/)
        HTTPError: an HTTP error occurred (https://docs.python-requests.org/en/master/_modules/requests/exceptions/)
    
    """

    headers = {
        'Content-Type': 'application/json',
        'Authorization': device["device"]["authentication"]["Ground_Token"]
    }

    # https://api.allthingstalk.io/swagger/ui/index
    url = "https://"+device["device"]["authentication"]["api"]+\
    "/device/"+device["device"]["authentication"]["Device_ID"]+\
    "/state"
        
    payload = {
        asset_name: {
            "value": payload_value,
            "at": payload_timestamp
        }
    }

    try:
        response = requests.request("PUT", url, headers=headers, json=payload)
        response.raise_for_status()
        message = 'ConnectionOK'
        return message
    except requests.exceptions.ConnectionError as exception:
        message = 'ConnectionError'
        return message
    except requests.exceptions.HTTPError as exception:
        message = 'HTTPError'
        return message
    
    
def get_asset_data_from_cloud(device, asset_name, payload_timestamp):
    """
    Obtain asset data from the Cloud

    Parameters
    ----------
    device: type dict
        device description (defined in a json file)
    asset_name: str
        name of the assets
    payload_timestamp: str
        timestamp

    Returns
    -------
    message: str or float
        data: the asset data 
        ConnectionError: a Connection error occurred. (https://docs.python-requests.org/en/master/_modules/requests/exceptions/)
        HTTPError: an HTTP error occurred (https://docs.python-requests.org/en/master/_modules/requests/exceptions/)
    
    """    
    
    # https://api.allthingstalk.io/swagger/ui/index
    url = "https://"+device["device"]["authentication"]["api"]\
    +"/device/"+device["device"]["authentication"]["Device_ID"]\
    +"/asset/"+asset_name+"/states?from="+payload_timestamp+"&page0"
    
    payload = {}
    headers = {
        'Authorization': device["device"]["authentication"]["Ground_Token"],
        'Content-Type': 'application/json'
    }

    try:
        response = requests.request("GET", url, headers=headers, data = payload)
        response.raise_for_status()
        return response.json()['data']
    except requests.exceptions.ConnectionError as exception:
        message = 'ConnectionError'
        return message
    except requests.exceptions.HTTPError as exception:
        message = 'HTTPError'
        return message
```

```Python
import datetime
import time
import json
import requests

with open('device.json') as json_file:
    device = json.load(json_file)
    
asset_name = 'CO2_ppm'

print('------------------')
print('Asset:',asset_name)
print('------------------')

start_datetime = datetime.datetime.utcnow().isoformat()

# Create a random list of payload values (risk values) between 1 and 10 (10 inclusive)
from random import randrange

payload = []
data_from_cloud = []
timestamp = []
label_from_cloud = []
value_from_cloud = []

payload = [1250,950,400,950,1250,950,400]
print('Payload:', payload)
print('Payload label:', decoder(payload))

# send & retrieve risk data
for value in payload:
    payload_timestamp = datetime.datetime.utcnow().isoformat()
    timestamp.append(payload_timestamp)
    print('Time:', payload_timestamp, '- Value:', value, ',- Label:', decoder([value]))
    
    # Send data
    send_asset_data_to_cloud(device, asset_name, payload_timestamp, value)
    time.sleep(5)
    
    # Retrieve data
    data_from_cloud.append(get_asset_data_from_cloud(device, asset_name, payload_timestamp))
    time.sleep(5)

print('------------------')
print('Data from Cloud.')
print('------------------')

data_from_cloud
```

```
------------------
Asset: CO2_ppm
------------------
Payload: [1250, 950, 400, 950, 1250, 950, 400]
Payload label: ['Bad', 'Moderate', 'Good', 'Moderate', 'Bad', 'Moderate', 'Good']
Time: 2021-08-27T09:13:54.643491 - Value: 1250 ,- Label: ['Bad']
Time: 2021-08-27T09:14:05.032053 - Value: 950 ,- Label: ['Moderate']
Time: 2021-08-27T09:14:15.410910 - Value: 400 ,- Label: ['Good']
Time: 2021-08-27T09:14:25.815638 - Value: 950 ,- Label: ['Moderate']
Time: 2021-08-27T09:14:36.196445 - Value: 1250 ,- Label: ['Bad']
Time: 2021-08-27T09:14:46.561765 - Value: 950 ,- Label: ['Moderate']
Time: 2021-08-27T09:14:56.907719 - Value: 400 ,- Label: ['Good']
------------------
Data from Cloud.
------------------

[[{'at': '2021-08-27T09:13:54.643Z', 'data': 1250}],
 [{'at': '2021-08-27T09:14:05.032Z', 'data': 950}],
 [{'at': '2021-08-27T09:14:15.41Z', 'data': 400}],
 [{'at': '2021-08-27T09:14:25.815Z', 'data': 950}],
 [{'at': '2021-08-27T09:14:36.196Z', 'data': 1250}],
 [{'at': '2021-08-27T09:14:46.561Z', 'data': 950}],
 [{'at': '2021-08-27T09:14:56.907Z', 'data': 400}]]
```

![AllThingsTalk_Pinboard_Test](.\img\Rules\AllThingsTalk_Pinboard_Test.PNG)

---

