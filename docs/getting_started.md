# Getting started

## Deployment

There are just a few strict rules to take into account when deploying the enodo platform. The Listener should be placed right next to you siridb instance, so it can listen to incoming data via a pipe. The worker can be deployed anywhere, as long as there is enough CPU resource to be used by the worker. The Hub should be deployed in such a way that it both the listener and the worker can connect to it via socket connections.


### Docker
**TODO**

### Localhost
Follow these steps for all the Enodo components:

1. Install dependencies via `pip3 install -r requirements.txt`
2. Setup a .conf file file `python3 main.py --create_config` There will be made a `default.conf` next to the main.py.
3. Fill in the `default.conf` file
4. Call `python3 main.py --config=default.conf` to start the hub.
5. You can also setup the config by environment variables. These names are identical to those in the default.conf file, except all uppercase.

Follow these additional steps for the Enodo Hub:

6. Fill in `settings.enodo` file, which you can find in the data folder by the path set in the conf file with key: `enodo_base_save_path`
7. Restart Hub to use new settings or fill them in via the GUI

## Enodo Hub API

The Enodo Hub has two API's from which you can do CRUD actions and subscribe to data changes. A REST API and a socket.io API.

### REST API

| Resource path | CRUD          |
| ------------- |:-------------:|
| /api/series      | CR |
| /api/series/{series_name}      | RD      |
| /api/enodo/event/output | CR      |
| api/enodo/event/output/{output_id} | D |



#### Examples

**Create Series**

call `{hostname}/api/series` (POST)
```
{
  "name": "series_name_in_siridb",
  "config": {
    "min_data_points": 2,
    "realtime": true,
    "job_config": [
      {
        "activated": true,
        "job_type": "job_base_analysis",
        "model": "prophet",
        "job_schedule_type": "points",
        "job_schedule": 200,
        "model_params": {
          "points_since": 1563723900,
          "sensitivity": 2,
          "static_rules": {
            "min": 800,
            "max": 1000,
            "last_n_points": 100
          }
        }
      },
      {
        "link_name": "anomaly_forecast",
        "activated": true,
        "job_type": "job_forecast",
        "model": "ffe",
        "job_schedule_type": "seconds",
        "job_schedule": 200,
        "model_params": {
          "points_since": 1563723900,
          "sensitivity": 2,
          "static_rules": {
            "min": 800,
            "max": 1000,
            "last_n_points": 100
          }
        }
      },
      {
        "link_name": "anomaly_forecast_test",
        "activated": true,
        "job_type": "job_forecast",
        "model": "prophet",
        "job_schedule_type": "seconds",
        "job_schedule": 200,
        "model_params": {
          "points_since": 1563723900,
          "sensitivity": 2,
          "static_rules": {
            "min": 800,
            "max": 1000,
            "last_n_points": 100
          }
        }
      },
      {
        "requires_job": "anomaly_forecast",
        "activated": true,
        "job_type": "job_anomaly_detect",
        "model": "ffe",
        "job_schedule_type": "seconds",
        "job_schedule": 200,
        "model_params": {
          "points_since": 1563723900,
          "sensitivity": 2,
          "forecast_name": "anomaly_forecast"
        }
      },
      {
        "requires_job": "anomaly_forecast_test",
        "activated": true,
        "job_type": "job_anomaly_detect",
        "silenced": true,
        "model": "ffe",
        "job_schedule_type": "seconds",
        "job_schedule": 200,
        "model_params": {
          "points_since": 1563723900,
          "sensitivity": 2,
          "forecast_name": "anomaly_forecast_test"
        }
      },
      {
        "requires_job": "anomaly_forecast",
        "activated": true,
        "job_type": "job_static_rules",
        "model": "static_rule_engine",
        "job_schedule_type": "seconds",
        "job_schedule": 200,
        "model_params": {
          "min": 800,
          "max": 1000,
          "last_n_points": 100,
          "n_predict": 100,
          "on_forecast": "anomaly_forecast"
        }
      }
    ]
  }
}
```


**Create event output stream**

call `{hostname}/api/enodo/event/output` (POST)
```
{
	"output_type": 1,
	"data": {
		"for_severities": ["warning", "error"],
		"url": "url_to_endpoint",
		"headers": {
			"authorization": "Basic abcdefghijklmnopqrstuvwxyz"
		},
		"payload": "{\n  \"title\": \"{{event.title}}\",\n  \"body\": \"{{event.message}}\",\n  \"dateTime\": {{event.ts}},\n  \"severity\": \"{{event.severity}}\"\n}"
	}
}
```



### Socket.IO Api (WebSockets)

When sending payload in a request, use the data structure same as in the REST API calls, except the data will be wrapped in an object : `{"data": ...}`.

#### Get series
event: `/api/series/create`

#### Get series Details
event: `/api/series/details`

#### Create series
event: `/api/series/create`

#### Delete series
event: `/api/series/delete`

#### Get all event output stream
event: `/api/event/output`

#### Create event output stream
event: `/api/event/output/create`

#### Delete event output stream
event: `/api/event/output/delete`


## Analysis
Enodo support the following analysis jobs:

1. Base series analysis for series characteristics
2. Forecasting
3. Anomaly detection
4. Statis rules

Each analysis job is send by the Hub to a available worker. The worker uses the series config to determine which model/algorithm to use for executing the job. Different workers can have different models implemented, which they let the Hub know while connecting on startup.

### 1. Base series analysis
This job is meant to gather series characteristics and simple data such as if the series has a trend of detectable seasonality in it.

### 2. Forecasting
The forecasting job results in a forecast of the series. The worker will use a requested model to forecast the series into the future, using the data we already have of this series. The forecast can differ between models and config. You can forecast just 5 hours into the future, or 5 days and so one. Depending on the amount of data and the model used for this job, it can be a very extensive or simple forecast.

### 3. Anomaly detection
Using a requested model/algorithm the worker will try to check if in the last *n* points, there were any anomalies within the data. The more suffisticated the model or algorithm, the more precise the worker can be.

### 4. Static rules
For simple series, a static threshold will do. For now Enodo support a min and max threshold.

### Analysis models
Models can be installed within the worker
```
analyser
├── model
    ├── __init__.py
    ├── models
    │   ├── base.py
    │   ├── ffe
    │   │   ├── __init__.py
    │   │   ├── bootstrap.py
    │   │   └── ffe.py
    │   └── prophet
    │       ├── __init__.py
    │       ├── bootstrap.py
    │       └── prophet.py
    └── util.py
```
Within the model folder you will find:

- util.py holding the load function, to load all model classes
- models folder holding all installed analyser models

When creating a new model, make sure that the modelclass itself extends the basemodel class in base.py, and that the bootstrap.py file will have a function `get_model_class` which will return the model class itself (not an instance)