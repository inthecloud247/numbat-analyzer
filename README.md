# numbat-dash

Dashboard/alert/monitor for the numbat-powered metrics system.

Information goes here.

## Development notes

Proposed design:

- There's a cluster of InfluxDBs.
- Each host runs a `hekad` configured for collection.
- Each service has a client that periodically sends interesting info to `hekad`.
- The per-host hekads send all data to InfluxDB.
- They also send it to the main collector hekad.
- The collector hekad rolls up/analyzes stats & feeds data to `numbat-dash`.
- `numbat-dash` is responsible for sending alerts & showing all *recent* information.
- If this service does its job, you delete your nagios installation.


Data flow:

- Incoming data ⇢ rules: display? alert?
- all data ⇢ in-memory data caches ⇢ short-term graphs
- some data ⇢ alerts
- alerts ⇢ outgoing integrations

Half of the above can happen in hekad. Hekad can then send the alerts/rollups to this app for display or other action. Separate responsibility: heka to analyze data, this app for display.

Implications:

- everything goes into InfluxDB: hekad output, operational actions, other human actions
- Dashboard needs to include both visual data (graphs) & current alert status
- data should probably get tagged with "how to display this" so a new stream of info from hekad can be displayed usefully sans config
- Dashboard should link to the matching Grafana historical data displays for each metric.
- CONSIDER: dashboard data displays *are* grafana, just of a different slice of influxdb data (rotated out regularly?) Dashboard page then becomes grafana with the alert stuff in an iframe or something like that. In this approach, the dashboard service is an extra-complex configurable set of hekad rules in javascript instead of Lua.
- Hello D3 or wrapper library?

This service:

- express web app for the page-serving part
- mostly it's a server that accepts data streams from hekad & processes them
- processing rules are javascript snippets
- probably a directory full of them that gets auto-reloaded? static on startup initially, though
- websockets/whatever to push updates from monitoring layer to the dashboard

Outgoing integrations:

- pagerduty
- slack messages

### What does a metric data point look like?

It must be a valid InfluxDB data point. Inspired by Riemann's events.

```javascript
{
    host: 'hostname.example.com',
    service: 'service.name',
    tags: ['array', 'of', 'tags'],
    status: 'okay' | 'warning' | 'critical' | 'unknown',
    description: 'textual description',
    time: ts-in-ms,
    ttl: ms-to-live,
    value: 42
}
```

Use tags to carry metadata. For instance: `[ 'event', 'deploy' ]` or `['counter']`.

### What do rules look like?

-- match
-- act

### What does a graph rule look like?

Looks for specific tags in event.

### What does an alert rule look like?

Looks for status. Also looks for expired data that should be present. (This seems weak. Develop.)

## License

ISC
