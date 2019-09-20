# Narratiive Public Data API

Narratiive Public Data API provides access to Narratiive Data through HTTP requests.

## Terminology

### Data Request 

Data Request is a request record of getting data. 

A Data Request contains Data Parameters that is to be sent to our data server; request dates (created/started/end), request status, ownerships. 
 

For example, we can say: 

 
> You can create a Data Request to get the data you need for your monthly report



### Data Parameters 


Data Parameters is the description to express the filters/time span/splits we use in requesting data. 

A Data Parameters contains information quite similar to the "Query" used in the Narratiive Dashboard, but less. It is mainly just the "Criteria" you picked in the query builder in the right sidebar:

<img src="https://user-images.githubusercontent.com/499870/65303077-4d624380-dbc0-11e9-854c-335f2213ecd8.png" width=300 />


For example, we will ask people: 

> What filters did you put in the Data Parameters for your Data Request". 

 
## How Does It Work

We provide several [http APIs](#endpoints) to help you get data from Narratiive, follow the steps below:

- Step1: Create A Data Request
    - Use our API to create a new data request with your specified data parameters, you will get the newly created data request id from the response.

- Step2: Get Result of A Data Request
    - Use the request id you get from the Step1 to get the current status and result of the data request process. Once it's status is "ready", you will get the result.
    
## API Authentication

The Public API uses secret token generated for each user to do the authentication.
 
### Get Your Token

Users should be able to get their token from the dashboard [My Settings](https://app.narratiive.com/settings/my_settings) page:

<img src="https://user-images.githubusercontent.com/499870/65302752-70d8be80-dbbf-11e9-818e-5aae18999a1a.png" width=300 />

<img src="https://user-images.githubusercontent.com/499870/65302800-94036e00-dbbf-11e9-8b5c-88ebb83cc940.png" width=300 />

If you can not find the setting, please contact our customer service for help.

### Use The Token

To use the token, for each public API call, just attach the token to the HTTP header: 

```
X-Narratiive-Data-API-Token: {Token} 

```

If authentication fails, a HTTP code 403 will be returned. 

## Endpoints

### Create A New Data Request 

- URL: `https://api.narratiive.com/public_api/data_requests`
- Method: `POST` 
- Parameters: 
    - `parameters`: string (a JSON string), required. See [Compose Data Parameters](/#compose-data-parameters) about how to get the parameters. 
- Response: Same as [Data Request Response](/#data-request-response).

#### Example:

```bash
curl -X POST \
  https://api.narratiive.com/public_api/data_requests \
  -H 'X-Data-API-Token: [replace with your token]' \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H 'cache-control: no-cache' \
  -H 'content-type: multipart/form-data' \
  -F 'parameters=["TIME_GRANULARITY/DAY","TIME_SPAN/LAST_30_DAYS","MEASURE/UB","SPLIT/PUBLICATION",{"type":"include","criteria":[]}]'
```


### Get All Data Requests 

- URL: `https://api.narratiive.com/public_api/data_requests`
- Method: `GET` 
- Parameters: n/a 
- Response: A array of Data Request object: 
    - `id`: string
    - `status`: string, values are "error", "updating", "scheduled", "ready"
    - `parameters`: The Data Parameters used in the data request
    - `created_at`: string.  

### Get Data Request Result 

- URL: `https://api.narratiive.com/public_api/data_requests/:request_id` 
- Method: `GET` 
- Parameters: n/a 
- Response: see "Data Request Response" 

#### Example:

```
curl -X GET \
  https://api.narratiive.com/public_api/data_requests/[DATA_REQUEST_ID] \
  -H 'X-Data-API-Token: [replace with your token]' \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H 'cache-control: no-cache' \
```

### Data Request Response 

Based on the status of a request, the response will have the following fields: 

- id: string 
- status: string, values are `error`, `updating`, `scheduled`, `ready` 
- data: the Data Result  
- error: Error message 

 
#### Data Result Format 


Example 1: Split by device type: 

```json
[
    {
        "items": [
            { 
                "key": "Mobile Phone", 
                "label": "Mobile Phone", 
                "measures": { 
                    "SAMPLE_SIZE": 10, 
                    "TB": 1133445, 
                    "UB": 9876655 
                } 
            } 
        ], 
        "measures": { 
            "SAMPLE_SIZE": 10, 
            "TB": 1133445, 
            "UB": 9876655 
        }, 
        "split_type": "DEVICE_TYPE" 
    }
] 
```

Example 2: Split by website and group by day: 

```json
[
  {
    "items": [
      {
        "Items": [
          {
            "Key": "2019-09-03T00:00:00.000Z",
            "label": "2019-09-03T00:00:00.000Z",
            "measures": {
              "SAMPLE_SIZE": 10,
              "TB": 1133445,
              "UB": 9876655
            }
          },
          ...
        ],
        "key": "2703",
        "label": "news24.com",
        "measures": {
          "SAMPLE_SIZE": 10,
          "TB": 1133445,
          "UB": 9876655
        }
      },
      ...
    ],
    "measures": {
      "SAMPLE_SIZE": 10,
      "TB": 1133445,
      "UB": 9876655
    },
    "split_type": "PUBLICATION"
  }
]
```

Example 3: Without split. 

```json
[
  {
    "measures": {
      "SAMPLE_SIZE": 10,
      "TB": 1133445,
      "UB": 9876655
    },
    "split_type": "DEVICE_TYPE"
  }
]
```


## Limitations

There will be a concurrency limits for how many Query Task can be run for each team.

The limitation works in forbidding team to create a new Query Task if the current running number of Query Task has reached the limit. 

The maximum concurrent data requests at the moment is 5 for each team.

## Compose Data Parameters

The easiest way to compose your data parameters is to use the Narratiive Dashboard query page as the Data Request Builder.

Use the query builder as you use it normally, after you picked all the filters and other criteria you want, open the "Public Data API" tab on the right sidebar, the `parameters` in the Curl command will be the parameters you need. 

<img src="https://user-images.githubusercontent.com/499870/65302899-e0e74480-dbbf-11e9-847e-98e661937600.png" width=300 />

