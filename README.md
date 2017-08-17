# mnubo SmartObjects Python client


[![Build status](https://travis-ci.org/mnubo/smartobjects-python-client.svg?branch=master)](https://travis-ci.org/mnubo/smartobjects-python-client)
[![PyPI](https://img.shields.io/pypi/v/smartobjects.svg?maxAge=2592000)](https://pypi.python.org/pypi/smartobjects/)


Table of Content
================

[1. Introduction](#1-introduction)

[2. At a glance](#2-at-a-glance)

[3. Requirements](#3-requirements)

[4. Installation & Configuration](#4-installation--configuration)

[5. Usage](#5-usage)

[6. Need help?](#6-need-help)

---

# 1. Introduction

This package provides a simple Python client to connect to mnubo's SmartObjects platform.
It gives access to the most common features of mnubo's REST API. All methods require proper authentication.
The MnuboClient object handles authentication under the hood based on the client_id and client_secret arguments.

Methods such as `create()`, `delete()`, etc, do not return any value. However if anything is invalid or goes wrong, an
exception will be thrown so you know what happened.


# 2. At a glance

_(optional arguments omitted)_

| Service | Method                                               | Summary                                                                 | Example                                           |
| ------- | ---------------------------------------------------- | ------------------------------------------------------------------------| ----------------------------------------          |
| Owners  | `create(owner)                                     ` | create a new owner                                                      | [simple_workflow.py](examples/simple_workflow.py) |
|         | `update(username, owner)                           ` | update an existing owner                                                |                                                   |
|         | `claim(username, device_id, optional_body)         ` | link an existing object to an existing owner                            | [simple_workflow.py](examples/simple_workflow.py) |
|         | `unclaim(username, device_id, optional_body)       ` | unlink an existing object from an existing owner                        | [simple_workflow.py](examples/simple_workflow.py) |
|         | `batch_claim(claims)                               ` | claim a batch of objects                                                | [simple_workflow.py](examples/simple_workflow.py) |
|         | `batch_unclaim(unclaims)                           ` | unclaim a batch of objects                                              | [simple_workflow.py](examples/simple_workflow.py) |
|         | `create_update(owners)                             ` | create or update a batch of owners                                      |                                                   |
|         | `delete(username)                                  ` | delete an owner                                                         |                                                   |
|         | `owner_exists(username)                            ` | check if an owner exists                                                | [simple_workflow.py](examples/simple_workflow.py) |
|         | `owners_exist(usernames)                           ` | check if a list of owners exist                                         |                                                   |
| Objects | `create(object)                                    ` | create a new smart object                                               | [simple_workflow.py](examples/simple_workflow.py) |
|         | `update(device_id, object)                         ` | update an existing object                                               |                                                   |
|         | `create_update(objects)                            ` | create or update a batch of objects                                     |                                                   |
|         | `delete(device_id)                                 ` | delete an object                                                        |                                                   |
|         | `object_exists(device_id)                          ` | check if an object exists                                               | [simple_workflow.py](examples/simple_workflow.py) |
|         | `objects_exist(device_ids)                         ` | check if a list of objects exist                                        |                                                   |
| Events  | `send(events)                                      ` | send a batch of events tagged with multiple devices                     |                                                   |
|         | `send_from_device(device_id, events)               ` | send an event tagged with a specific device                             | [simple_workflow.py](examples/simple_workflow.py) |
|         | `event_exists(event_id)                            ` | check if an event exists                                                |                                                   |
|         | `events_exist(event_ids)                           ` | check if list of events exist                                           |                                                   |
| Search  | `search(query)                                     ` | performs a search in the platform with the provided JSON query (MQL)    | [simple_workflow.py](examples/simple_workflow.py) |
|         | `validate_query(query)                             ` | validates a MQL query                                                   | [simple_workflow.py](examples/simple_workflow.py) |
|         | `get_datasets()                                    ` | retrieves the list of datasets available for this account               | [simple_workflow.py](examples/simple_workflow.py) |
| Model   | `export()                                          ` | fetches the model in the current zone                                   | [simple_workflow.py](examples/simple_workflow.py) |



---
# 3. Requirements

- Python 2.7
- libraries: `requests`, `six`


---
# 4. Installation & Configuration

From [PyPI](https://pypi.python.org/pypi/smartobjects/):

    $ pip install smartobjects

Alternatively, if you want to use the client from a data science perspective you can install with `pandas` extra (see [Search Services](###use-the-search-services):

    $ pip install smartobjects[pandas]

From the sources:

    $ git clone https://github.com/mnubo/smartobjects-python-client.git
    $ cd smartobjects-python-client
    $ python setup.py install

---
# 5. Usage

### Initialize the MnuboClient

```python
from smartobjects import SmartObjectsClient
from smartobjects import Environments
from smartobjects.smartobjects_client import ExponentialBackoffConfig

client = SmartObjectsClient('<CLIENT_ID>', '<CLIENT_SECRET>', Environments.Production)
```

The environment argument can be `Environments.Sandbox` or `Environments.Production` and automatically resolves to the right
API URL.

_Optional arguments_:

- compression_enabled: if `True`, data sent to the platform is compressed using _gzip_ format. Default: `True`
- backoff_config: if given, requests resulting in 503 will be retried. Default: `None`

_Note:_ to use exponential backoff retries feature, you must `pip install tenacity` (4.2.0+)

### Use the Owners service
To create owners on the mnubo SmartObjects platform, please refer to
the data modeling guide to format correctly the owner's data structure.

#### Create an Owner
```python
client.owners.create({
    "username": "sheldon.cooper@caltech.edu",
    "x_password": "****************",
    "zip_code": "91125"
})
```

_Mandatory properties_: `username`, `x_password`

#### Claim or unclaim a Smart Object for an Owner
```python
client.owners.claim('sheldon.cooper@caltech.edu', 'fermat1901')
# if you want to override some values
client.owners.claim('sheldon.cooper@caltech.edu', 'fermat1901', {
    "x_timestamp": 2015-02-01T05:00:00.000Z"
})


client.owners.unclaim('sheldon.cooper@caltech.edu', 'fermat1901')
# if you want to override some values
client.owners.unclaim('sheldon.cooper@caltech.edu', 'fermat1901', {
    "x_timestamp": 2015-02-01T05:00:00.000Z"
})
```

As a batch:
```python
client.owners.batch_claim([
    ('sheldon.cooper@caltech.edu', 'fermat1901'), 
    ('leonard.hofstadter@caltech.edu', 'ramanujan1887')
])
```

#### Update an Owner
```python
client.owners.update('sheldon.cooper@caltech.edu', {
    "zip_code": "94305",    # update of an existing property
    "service_type": "phd"   # creation of a new property
})
```
#### Create or update owners in batch
```python
results = client.owners.create_update([
    {"username": "sheldon.cooper@caltech.edu", "service_type": "prof"},
    {"username": "leonard.hofstadter@caltech.edu", "x_password": "*******"}   
])
```
_Mandatory properties_: `x_username_id` (all owners), `x_x_password_type` (new owners)

Returns a list of `Result` with the completion status of each operation (and reason of failure if any).

#### Delete an Owner
```python
client.owners.delete('sheldon.cooper@caltech.edu')
```

#### Check if an owner exists

```python
>>> client.owners.owner_exists('sheldon.cooper@caltech.edu')
True

>>> client.owners.owners_exist(['sheldon.cooper@caltech.edu', 'hwolowitz.phd@caltech.edu'])
{'sheldon.cooper@caltech.edu': True, 'hwolowitz.phd@caltech.edu': False}
```


### Use the Smart Objects Service
To create smart objects on the mnubo SmartObjects platform, please refer to
the data modeling guide to format correctly the smart object's data structure.

#### Create a Smart Object
```python
client.objects.create({
    "x_device_id": "fermat1901",
    "x_object_type": "calculator",
    "precision": "infinite"
})
```
_Mandatory properties_: `x_device_id`, `x_object_type`

#### Update a Smart Object
```python
client.objects.update("fermat1901", {
    "is_valid": True
})
```

#### Create or update objects in batch

If an object doesn't exist, it will be created, otherwise it will be updated.

```python
client.objects.create_update([
    {"x_device_id": "fermat1901", "is_valid": False},
    {"x_device_id": "ramanujan1887", "x_object_type": "pie"}
])
```
_Mandatory properties_: `x_device_id` (all objects), `x_object_type` (new objects)

Returns a list of `Result` objects with the completion status of each operation (and reason of failure if any).

#### Delete a Smart Object
```python
client.objects.delete("fermat1901")
```

#### Check if an object exists

```python
>>> client.objects.object_exists('fermat1901')
True

>>> client.objects.objects_exist(['fermat1901', 'teleporter'])
{'fermat1901': True, 'teleporter': False}
```


### Use the Event Services
To send events to the mnubo SmartObjects platform, please refer to
the data modeling guide to format correctly the event's data structure.

#### Send an Event
```python
results = client.events.send([
    {"x_object": {"x_device_id": "fermat1901"}, "status": "running"},
    {"x_object": {"x_device_id": "ramanujan1887"}, "ratio": 3.1415}
])
```

_Optional arguments_:
- `must_exist`: if `True`, an event referring an unknown object will be rejected (default to `False`)
- `report_results`: if `True`, a list of `EventResult` objects will be returned with the status of each operation.
      If `False`, nothing will be returned when _all_ events are successfully ingested, but a `ValueError` exception
      will be thrown if at least one fail. Default to `True`.

#### Send an event tagged with a device

This method allows sending multiple events for a given device without the need of setting the target in the payload.

```python
results = client.events.send_from_device("ramanujan1887", [
    {"ratio": 3.1414},
    {"ratio": 3.1413}
])
```

_Optional arguments_:
-   `report_results`: if `True`, a list of `EventResult` objects will be returned with the status of each operation.
    If `False`, nothing will be returned when _all_ events are successfully ingested, but a `ValueError` exception
    will be thrown if at least one fail. Default to `True`.


#### Check if an event already exists

```python
>>> client.events.event_exists(UUID("1ff58794-f0da-4738-8444-68a592de6746"))
True

>>> client.events.events_exist([UUID("1ff58794-f0da-4738-8444-68a592de6746"), uuid.uuid4()])
{UUID("1ff58794-f0da-4738-8444-68a592de6746"): True, UUID("e399afda-3c8b-4a6d-bf9c-c51b846c214d"): False}
```


### Use the Search Services
To send search queries to the mnubo SmartObjects platform, please refer to
the Search API documentation to format your queries correctly.

#### Search Query
```python
resultset = client.search.search({
    "from": "owner",
    "limit": 100,
    "select": [
        {"value": "username"}
    ]
})
```

This method returns a `ResultSet` containing the result of the search (columns and rows).

```python
>>> "Got {} results!".format(len(resultset))
Got 2 results!
>>> [row.get("username") for row in resultset]
["sheldon.cooper@caltech.edu", "leonard.hofstadter@caltech.edu"]
```

Optionally, if you are using the SDK with [pandas](http://pandas.pydata.org/pandas-docs/stable/index.html), you can use the propery `df` of the `resultset`. It will return a DataFrame version of the results, with `datetime` columns converted to a [datetime](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.to_datetime.html#pandas.to_datetime) structure.

```python
resultset = client.search.search({
    "from": "event",
    "select": [
        {"count": "*"}
    ],
    "groupByTime": {
        "interval": "year",
        "field": "x_timestamp"
    }
})

resultset.df
#                   year  COUNT(*)
# 0 2017-01-01 05:00:00      1232
```

#### Validate a search query

For complex queries, it is recommended to use this feature to reduce errors.

```python
validation_result = client.search.validate_query({
    "invalid": "owner",
    "limit": 100,
    "select": [
        {"value": "username"}
    ]
})
```

This method returns a `QueryValidationResult` object with the status of the validation and list of errors if any:

```python
>>> validation_result.is_valid
False
>>> validation_result.validation_errors
["a query must have a 'from' field"]
```


#### Retrieve namespace datasets

This method allows to retrieve the different datasets available for querying. Result is a map of `DataSet` objects
indexed by the dataset name (`owner`, `object`, `event`, `session`).

```python
>>> datasets = client.search.get_datasets()
>>> [event.key for event in datasets['event'].fields]
["event_id", "x_object.x_device_id", "timestamp", ...]
```

### Use the Model Service

To retrieve the model as it is currently in the zone you are working (sandbox or production), you can use
the ModelService:

```python
model = client.model.export()
print(len(model.eventTypes)) # outputs: 2
```



# 6. Need help?

Reach us at support@mnubo.com
