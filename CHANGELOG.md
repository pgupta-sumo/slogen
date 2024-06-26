### [v1.3](https://github.com/OpenSLO/slogen/releases/tag/v1.3)

- **Fix** : Prevents reordering of slo tags on generating terraform configuration


### [v1.2](https://github.com/OpenSLO/slogen/releases/tag/v1.2)

#### Support for Labels
`labels` from the OpenSLO config are now applied as tags to the native SLOs created in the vendor environment (Sumologic). In the OpenSLO spec, `metadata.labels` is an optional map of keys to one or more values. Slogen supports only one value per unique key. The first value specified in the list of values will be considered.


### [v1.0](https://github.com/OpenSLO/slogen/releases/tag/v1.0.0)

- **Feature** : Support for OpenSLO v1 spec for `SLO`, `AlertPolicy` and `AlertNotificationTarget`
  - `AlertPolicy` and `AlertNotificationTarget` based approach removes the need for created duplicate alerting configs with each `SLO`
- **Feature** : Support for sumologic native SLO & SLO based monitors.
- **Feature** : Generates flow graph of which `SLO` are using which `AlertPolicy` and `AlertNotificationTarget`

![dep-graph](misc/samples-flowchart.png)

sample configs to try out the above features are available [here](samples/sumologic/v1)



### [v0.7](https://github.com/OpenSLO/slogen/milestone/2?closed=1)

- **Feature** : Timeslice based budgeting (availability calculated w.r.t. good time windows)  
  - [Sample config](samples/openslo/ingest-lag-timeslice-budgeting.yaml)
  
- **Feature** : Track and filter your SLO by fields e.g. customerID, region etc
  - [Sample config](samples/openslo/ingest-lag-timeslice-budgeting.yaml)
  - [Screenshot](misc/SLO-breakdow.png)

- **Feature** : Overview of all SLO's configured for each service
  - [Screenshot](misc/service-overview.png)
  
- **Feature** : SLO budget forecasting
  - [Screenshot](misc/budget-forecast.png)

- **Feature** : subcommand to list connection id for use in alert notification field
  - `slogen list -c`

- **Fix** : "resource not found error" on changing service name for existing SLO

- **Fix** : tool unable to run behind https proxy

#### Additional details

##### What is `timeslice` based budgeting ?
In timeslice budgeting instead of using the good vs bad request in the entire compliance period, the reliability is calculated as the amount of time the service was running at an acceptable level. For e.g. we define the service is up if in a 1 minute window `error rate < 1%` or `p90(latency) < 500ms`. In that case availability in a month is calculated as `(number of minutes when service was up)/(total number of minutes in month)`.

The advantage of `Timeslices` is that it aligns well with the concept of `uptime` which is easier to track on a regular basis 
i.e. instead of “how many shopping cart failures did we have',' we track “for how many minutes were shopping carts offline?“.
This better correlates to business outcomes where customers are very likely to retry an event after some time when they encounter failure / slowness or for async jobs that will be reattempted again and can afford to have some delay.  

`Occurrences` based budgeting on the other hand are more accurate, since it automatically weights your traffic based on throughput.  
If you have more traffic in the middle of the day, that’s more likely to influence your SLO performance than your low-traffic hours in the middle of the night.  
Similarly, if you have a few minutes of downtime, but those minutes served a large percentage of your traffic for the day, your burn rate will spike accordingly.

A detailed comparison with actual computation involved can be found [here](https://gist.github.com/nobl9-mikec/a1a55d97d77f10216be775eaad7221ac)

(Examples taken from [discussion](https://openslo.slack.com/archives/C0202J83M3R/p1637255459106800?thread_ts=1637242125.103900&cid=C0202J83M3R) in OpenSLO slack forum)
##### How is SLO tracking by fields done ?
For e.g. if we specify `customerID` as a field for SubSLI breakdown, 
then if `customerA` has 10 messages and `customerB` has 20 messages in the compliance window, `customerA` has 4 slow messages 
and `customerB` has 2 slow messages, then the `SLI_A` = 60% and `SLI_B` = 90% and `SLI_overall` = 80%. 

The same is true for timeslice where the goodness/uptime for that timeslice for a given customer is based on 
requests for that customer only but overall goodness for a given timeslice is calculated by looking at the requests for all customers. 
For e.g. if we specify a `timesliceTraget` as 0.75 (or that 75% of requests in a timeslice should be good) 
then for the same distribution of requests in a single timeslice window, service was up for `customerB` but down for `customerA` but up overall.

