# Welcome to the SiriDB Enodo docs

For sourcecode please visit the [SiriDB Enodo Hub repo](https://github.com/SiriDB/siridb-enodo-hub).

## About SiriDB Enodo
Enodo is a time series data analysis platform made for SiriDB. Combining the power of storing and querying time series data from SiriDB and the analyzing power of Enodo, we can create better understanding of the data that we collect and store. So we can learn from the past and create forecasts for the future.

The Enodo platform is build in modules to create scalability. The Hub will control and organize the data that we have collected and the questions we want to ask about the data. The Worker will perform the analyzing jobs and the Listener will stay on top of the latest data points. Together these components will make sure we can monitor the data in realtime, adjust our expectations for the future and watch for sudden unexpected changes in the data that we collect.
Listener

The Enodo listener listens to a pipe socket with SiriDB server. The listener only keeps track of series that are registered via the hub. It sums up the added data points to each of these series and it sends periodically an update to the hub, or when a serie is monitored in realtime, it will notify the Hub immediately. The listener is separated from the Enodo hub, so that it can be placed close to the SiriDB server to gain local access to the pipe socket.
Worker

*Note : A worker uses significant CPU and thus should be placed on a machine that has low CPU usage.*

The Enodo worker executes fitting and forecasting models/algorithms. It can create different models like MA/RA/ARIMA, but also use Prophet, for analyzing series, training models with new data, flagging anomalies and calculate forecasts for a certain series. A worker can implement multiple models. This can be different per worker version. The implemented models should be communicated to the hub during the handshake.
Hub

The Enodo hub communicates and guides both the listener and the worker. It tells the listener to which series it needs to pay attention to, and it tells the worker which series should be analysed. Clients can connect to the hub for receiving updates, and polling for data. Also a client can use the hub to alter the data about which series should be watched.

## Deployment

There are just a few strict rules to take into account when deploying the enodo platform. The Listener should be placed right next to you siridb instance, so it can listen to incoming data via a pipe. The worker can be deployed anywhere, as long as there is enough CPU resource to be used by the worker. The Hub should be deployed in such a way that it both the listener and the worker can connect to it via socket connections.

## Getting started

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
		"job_models": {
			"job_base_analysis": "prophet",
			"job_forecast": "prophet",
			"job_anomaly_detect": "prophet"
		},
		"job_schedule": {
			"job_base_analysis": 200
		},
		"min_data_points": 2,
		"model_params": {
			"points_since": 1563723900
		}
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