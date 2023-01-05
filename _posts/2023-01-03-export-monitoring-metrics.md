---
title: Collecting GCP Monitoring Metrics 
tags: GCP, Docker, Terraform, Python
---

Have you ever used the metrics explorer in the GCP Monitoring service? It displays the metric data in time series format for various services like storage, pubsub etc.
I needed this data on one of my dashboards during an internship. I thought there would be an easy way to export this data, but boy was I wrong.
A senior pointed me to this resource -> [Reading metric data](https://cloud.google.com/monitoring/custom-metrics/reading-metrics) so I figured that the only way to collect metric data is by requesting the Monitoring API.

So I came up with a python script that does it (simplified) : 

```python
import os
import logging
import time
from flask import Flask
from google.cloud import bigquery, monitoring_v3

app = Flask(__name__)

def get_metric_data(project_id: str, metric_type: str, metric_name: str):
    client = monitoring_v3.MetricServiceClient()
    project_name = f"projects/{project_id}"
    interval = monitoring_v3.TimeInterval()

    now = time.time()
    seconds = int(now)
    nanos = int((now - seconds) * 10 ** 9)
    interval = monitoring_v3.TimeInterval(
        {
            "end_time": {"seconds": seconds, "nanos": nanos},
            "start_time": {"seconds": (seconds - 86400), "nanos": nanos},
        }
    )

    aggregation = monitoring_v3.Aggregation(
        {
            "alignment_period": {"seconds": 86400},
            "per_series_aligner": monitoring_v3.Aggregation.Aligner.ALIGN_MAX,
        }
    )

    results = client.list_time_series(
        request={
            "name": project_name,
            "filter": f'metric.type = "{metric_type}"',
            "interval": interval,
            "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL,
            "aggregation": aggregation
        }
    )

    if not results._response.time_series:
        data = [
            {"MetricType": metric_name,
             "Time": end_time.strftime('%Y-%m-%d %H:%M:%S'),
             "MetricValue": 0} ]
    else:
        for result in results:
            data = [
                {"MetricType": metric_name,
                    "Time": end_time.strftime('%Y-%m-%d %H:%M:%S'),
                    "MetricValue": result.points[0].value.int64_value}
            ]

    return data


def load_metric_data(project_id: str, dataset: str, table:str, data):
    client = bigquery.Client(project = project_id)
    table_ref = "{}.{}".format(dataset, table)
    table = client.get_table(table_ref)
    errors = client.insert_rows(table, data)
    if not errors:
        print("New rows have been added !")
    else:
        print("Encountered errors while inserting rows: {}".format(errors))


@app.route('/', methods=['POST'])

def handle_post():
    data = get_metric_data(
        project_id="silicon-synapse-372206",
        metric_type="storage.googleapis.com/storage/total_bytes",
        metric_name="bucket_total_bytes"
        )
    load_metric_data(
        project_id="silicon-synapse-372206",
        dataset="metrics",
        table="metrics-table",
        data=data
    )
    

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=int(os.environ.get('PORT', 8080)))
```

If you notice its just 2 functions `get_metric_data()` to request the metric data from monitoring API, and `load_metric_data()` to load the response to a Big Query table.
And I wrapped the whole thing into a flask application beacuse I wanted to schedule this script on Cloud run as it can scale to zero, and cloud run [requires](https://cloud.google.com/run/docs/developing) the container to be a webapp like flask app.

![architecture](https://github.com/nimblefox/GCP-Monitoring-Metrics/blob/main/GCP%20Architecture.png)

Next I had to send a post request with a cloud scheduler but due to sime ingress setting sin the project that doesnt let scheduler to trigger cloud run, I had to add a workflow to create the POST request

```
- call:
    try:
        call: http.post
        args:
            url: https://metrics-collector-xxxxxxxxxx-uc.a.run.app/
            auth:
                type: OIDC
        result: floor_result
    retry: ${http.default_retry}
- return_result:
    return: ${floor_result}
```

<!--more-->

---

You can find the code on my github --> [GCP-Monitoring-Metrics](https://github.com/nimblefox/GCP-Monitoring-Metrics)
