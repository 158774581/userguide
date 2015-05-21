Once a deployment is created and running, it is important to be able to monitor
its behavior and make sure it is operating as expected. With GraphLab Create, we
offer the tools needed to monitor and operate Predictive Service deployments.

##### Predictive Service Dashboard in GraphLab Canvas

To visualize a deployment using GraphLab Canvas, simply run
[show](https://dato.com/products/create/docs/generated/graphlab.deploy._predictive_service._predictive_service.PredictiveService.show.html#graphlab.deploy._predictive_service._predictive_service.PredictiveService.show),
as follows:

```no-highlight
deployment.show()
```

[<img alt="Example of a Predictive Service Deployment in GraphLab Canvas" src="images/predictive-services-dashboard-glc1.1.png" style="max-width: 70%; margin-left: 15%;" />](images/predictive-services-dashboard-glc1.1.png)

This will launch GraphLab Canvas in your browser and show you a dashboard for
this Predictive Service, like the example above.

If you would like to see an overall dashboard of all your Predictive Services,
run:

```no-highlight
graphlab.deploy.predictive_services.show()
```

Using the Predictive Service dashboard you can see the important metrics about
the deployment for the last six hours.

##### Getting Overall Deployment Status

To retrieve the overall status of a deployment, call
[get_status](https://dato.com/products/create/docs/generated/graphlab.deploy._predictive_service._predictive_service.PredictiveService.get_status.html#graphlab.deploy._predictive_service._predictive_service.PredictiveService.get_status).

```no-highlight
print deployment.get_status()
```

You may get status of different aspects of the deployment by way of the `view`
parameter to `get_status`

```no-highlight
# get node status
deployment.get_status(view='node')
```

```
Columns:
	cache	str
	dns_name	str
	instance_id	str
	state	str

Rows: 1

Data:
+---------+-------------------------------+-------------+-----------+
|  cache  |            dns_name           | instance_id |   state   |
+---------+-------------------------------+-------------+-----------+
| Healthy | ec2-52-24-53-250.us-west-2... |  i-8af8867c | InService |
+---------+-------------------------------+-------------+-----------+
[1 rows x 4 columns]
```

Of alternatively,

```no-highlight
# get cache status
deployment.get_status(view='cache')

# get Predictive Object status
deployment.get_status(view='model')
```

This API returns an SFrame regarding each Predictive Object's status on each
node in the deployment, so it is easy to verify programmatically when a
Predictive Object has been fully loaded by all nodes in the deployment.

##### Recovering from node failure

Occasionally, Predictive Service nodes fail. How would you discover this problem
in the first place? Most likely, you’ve found that occasionally queries are
timing out. Or better yet, you have configured
[CloudWatch](http://aws.amazon.com/cloudwatch/) metrics monitoring for your
service and you received an alert indicating that something is wrong. You can
manually inspect the status of the deployment with the `get_status` method. If
you find that a node is unreachable ("Unable to connect"), you may decide to
replace the node using
[replace_nodes](https://dato.com/products/create/docs/generated/graphlab.deploy._predictive_service._predictive_service.PredictiveService.replace_nodes.html#graphlab.deploy._predictive_service._predictive_service.PredictiveService.replace_nodes).

```no-highlight
deployment.replace_nodes(['i-8af8867c'])
```

```
[INFO] Adding 1 nodes to cluster
[INFO] Launching an m3.xlarge instance in the us-west-2a availability zone, with
       id: i-c5dda333. You will be responsible for the cost of this instance.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
^[[D[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster not fully operational yet, [1/2] instances currently in service.
[INFO] Cluster is fully operational, [2/2] instances currently in service.
[INFO] Terminating EC2 host(s) ['i-8af8867c'] in us-west-2
```

This method terminates each of the specified nodes and brings up new ones in
their place with the appropriate settings for your Predictive Service
deployment.

Note: currently we cannot preserve the state of the distributed cache when
repairing the cluster. In other words, all data in the cache at the time repair
is called will be lost.

Similarly, you can add and remove nodes with the
[add_nodes](https://dato.com/products/create/docs/generated/graphlab.deploy.PredictiveService.add_nodes.html?highlight=add_nodes#graphlab.deploy.PredictiveService.add_nodes)
and
[remove_nodes](https://dato.com/products/create/docs/generated/graphlab.deploy.PredictiveService.remove_nodes.html?highlight=remove_nodes#graphlab.deploy.PredictiveService.remove_nodes)
APIs. 

##### Cache Management

Caching is supported for Predictive Service clusters of any size. It can be
controlled both at the cluster level and at the Predictive Object level. By
default, the cache is enabled globally (ie. for all models) in a Predictive
Service deployment. Caching is enabled for all Predictive Objects by default.
You may control the cache globally and at the level of individual Predictive
Objects.

You may enable/disable/clear the cache at the cluster level using the following
commands:

```no-highlight
# enable cache for the cluster
# note this will turn on cache for all Predictive Objects
deployment.cache_enable()

# disable cache for the cluster
# note this will turn off cache for all Predictive Objects
deployment.cache_disable()

# clear all cache entries in the cluster
deployment.cache_clear()
```

You may turn the cache on/off for individual Predictive Object, which overrides
the global setting:

```no-highlight
# disable cache for a Predictive Object called 'my-no-cache-model'
deployment.cache_disable('my-no-cache-model')

# enable cache for a Predictive Object called 'my-no-cache-model'
deployment.cache_enable('my-no-cache-model')
```

##### CORS support

To enable CORS support for cross-origin requests coming from a website
(ex. 'https://dato.com'), use set_CORS:

```no-highlight
# To allow requests coming from https://dato.com
deployment.set_CORS('https://dato.com')
```

To enable CORS support for all cross-origin requests:

```no-highlight
# allow any CORS request
deployment.set_CORS('*')
```

To disable CORS support for this Predictive Service:

```no-highlight
# disable CORS support
deployment.set_CORS()
```

##### View Deployment Metrics

To zoom in beyond the basic dashboard, you can click through to Amazon
CloudWatch, where the metrics for a Predictive Service are stored. Using the
dashboard offered there you can see the individual metrics for a Predictive
Object, or overall metrics about the deployment.

##### Setting Alarms using CloudWatch Alarms

To set alarms for a deployment, use the Amazon CloudWatch console for the
metrics published by the Predictive Service Deployment.

For more information see the
[Amazon CloudWatch Alarms](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/AlarmThatSendsEmail.html)
documentation.

##### Reconfigure a Predictive Service Deployment

You can modify some underlying configuration parameters of the Predictive
Service deployment using the
[reconfigure](graphlab.deploy.PredictiveService.reconfigure.html?highlight=reconfigure#graphlab.deploy.PredictiveService.reconfigure)
method. At present, only two underlying configuration parameters have been
exposed, but this list will grow over time. The list below enumerates the
available configuration options:

- `cache_max_memory_mb`: the amount of memory allocated to the distributed cache
- `cache_ttl_on_update_secs`: the TTL of existing cache keys after a model update

An example call to reconfigure might look as follows:

```no-highlight
    # increase the cache_max_memory size to 4 GBs
    deployment.reconfigure({"cache_max_memory_mb": 4000})
```

##### Terminating a Predictive Service Deployment

To terminate a Predictive Service, call the
[terminate_service](https://dato.com/products/create/docs/generated/graphlab.deploy._predictive_service._predictive_service.PredictiveService.terminate_service.html#graphlab.deploy._predictive_service._predictive_service.PredictiveService.terminate_service)
method. There are options to delete the logs and Predictive Objects as
well. **Note:** There is no warning or confirmation on this method; it will
terminate the EC2 instances and teardown the Elastic Load Balancer.

```no-highlight
deployment.terminate_service()
```
