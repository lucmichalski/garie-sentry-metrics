# Garie sentry-metrics plugin

<p align="center">
  <p align="center">Tool to gather statistics from sentry and matomo, and supports CRON jobs.<p>
</p>

**Highlights**

-   Poll for sentry and matomo statistics and stores the data into InfluxDB
-   View all historic reports.
-   Setup within minutes

## Overview of garie-sentry-metrics

Garie-sentry-metrics was developed as a plugin for the [Garie](https://github.com/boyney123/garie) Architecture.

[Garie](https://github.com/boyney123/garie) is an out the box web performance toolkit, and `garie-securityheaders` is a plugin that generates and stores securityheaders data into `InfluxDB`.

`Garie-sentry-metrics` can also be run outside the `Garie` environment and run as standalone.

If your interested in an out the box solution that supports multiple performance tools like `securityheaders`, `google-speed-insight` and `lighthouse` then checkout [Garie](https://github.com/boyney123/garie).

If you want to run `garie-sentry-metrics` standalone you can find out how below.

## Getting Started

### Prerequisites

-   Docker installed

### Running garie-sentry-metrics

You can get setup with the basics in a few minutes.

First clone the repo.

```sh
git clone https://github.com/eea/garie-sentry-metrics.git
```

Next setup you're config. Edit the `config.json` and add websites to the list.

```javascript
{
  "plugins":{
        "sentry-metrics":{
            "cron": "0 */4 * * *"
        }
    },
  "cron": "0 */4 * * *",
  "urls": [
    {
      "url": "www.test1.com",
      "plugins": {
        "sentry-metrics":{
          "sentry_config": [
            {
              "sentryId": 23
            }
          ],
          "matomoId": 12
        }
      }
    },
    {
      "url": "www.test2.com",
      "plugins": {
        "sentry-metrics":{
          "sentry_config": [
            {
              "sentryId": 23,
              "filters":{
                "jsEvents":{
                        "operator":"equals", "field":"tags.app_name", "value":"global-search"
                }
              }
            }
          ],
          "matomoId": 12
        }
      }
    },
    {
      "url": "www.test3.com",
      "plugins": {
        "sentry-metrics":{
          "sentry_config": [
            {
              "sentryId": 23,
              "filters":{
                "jsEvents":{
                        "operator":"equals", "field":"tags.app_name", "value":"CaR"
                }
              }
            }
          ],
          "matomoId": 12
        }
      }
    },
    {
      "url": "www.test4.com",
      "plugins": {
        "sentry-metrics":{
          "sentry_config": [
            {
              "sentryId": 23,
              "filters":{
                "jsEvents":{
                        "operator":"or", "operands":[{"operator":"equals", "field":"tags.app_name", "value":"global-search"}, {"operator":"equals", "field":"tags.app_name", "value":"CaR"}]
                }
              }
            }
          ],
          "matomoId": 12
        }
      }
    },
    {
      "url": "www.test5.com",
      "plugins": {
        "sentry-metrics":{
          "sentry_config": [
            {
              "sentryId": 23,
              "filters":{
                "jsEvents":{
                        "operator":"not", "operand":{"operator":"or", "operands":[{"operator":"equals", "field":"tags.app_name", "value":"global-search"}, {"operator":"equals", "field":"tags.app_name", "value":"CaR"}]}
                }
              }
            }
          ],
          "matomoId": 12
        }
      }
    },
    {
      "url": "www.test6.com",
      "plugins": {
        "sentry-metrics":{
          "sentry_config": [
            {
              "sentryId": 23,
              "filters":{
                "jsEvents":{
                        "operator":"not", "operand":{"operator":"equals", "field":"tags.app_name", "value":"global-search"}
                }
              }
            }
          ],
          "matomoId": 12
        }
      }
    }
  ]
}
```

Once you finished edited your config, lets setup our environment.

```sh
docker-compose up
```

This will build your copy of `garie-sentrymetrics` and run the application.

On start garie-uptimerobot will start to gather statistics for the websites added to the `config.json`.

## config.json

| Property | Type                | Description                                                                          |
| -------- | ------------------- | ------------------------------------------------------------------------------------ |
| `cron`   | `string` (optional) | Cron timer. Supports syntax can be found [here].(https://www.npmjs.com/package/cron) |
| `urls`   | `object` (required) | Config for sentrymetrics. More detail below                                          |

**urls object**

| Property                                                       | Type                 | Description                                                                          |
| -------------------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------ |
| `url`                                                          | `string` (required)  | Url to get uptimerobot metrics for.                                                  |
| `plugins`                                                      | `object` (optional)  | To setup custom configurations.                                                      |
| `plugins.sentry-metrics`                                       | `object` (required)  | To setup custom sentry-metrics config.                                               |
| `plugins.sentry-metrics.matomoId`                              | `number` (required)  | Matomo project Id                                                                    |
| `plugins.sentry-metrics.sentry_config`                         | `list` (required)    | List of sentry configurations                                                        |
| `plugins.sentry-metrics.sentry_config[n].sentryId`             | `number` (optional)  | Unique sentry id. If not specified, *organizationSlug* and *sentrySlug* are required |
| `plugins.sentry-metrics.sentry_config[n].organizationSlug`     | `string` (optional)  | sentry organization name                                                             |
| `plugins.sentry-metrics.sentry_config[n].sentrySlug`           | `string` (optional)  | sentry project name                                                                  |
| `plugins.sentry-metrics.sentry_config[n].filters`              | `json` (optional)    | Describe how to filter sentry messages                                               |
| `plugins.sentry-metrics.sentry_config[n].filters.jsEvents`     | `json` (optional)    | Describe filters for js events                                                       |
| `plugins.sentry-metrics.sentry_config[n].filters.serverEvents` | `json` (optional)    | Describe filters for server events                                                   |


### Configuring the filters
For configuring the filters we have to build a json object
Operators can be:
equals, contains, startsWith, or, and, not
equals, contains, startsWith has 2 operands: field and value
ex:
```javascript
{
    "operator":"equals",
    "field":"tags.app_name",
    "value":"global-search"
}
```
or, and has a list of "subfilters"
ex: 
```javascript
{
    "operator":"or",
    "operands":[
        {
            "operator":"equals",
            "field":"tags.app_name",
            "value":"global-search"
        },
        {
            "operator":"equals",
            "field":"tags.app_name",
            "value":"CaR"}
    ]
}
```
not has a subfilter
ex:
```javascript
{
    "operator":"not",
    "operand":{
        "operator":"equals",
        "field":"tags.app_name",
        "value":"global-search"
    }
}
```

This way it's possible to create complex filters like:
```javascript
{
    "operator":"not",
    "operand":{
        "operator":"or",
        "operands":[
            {
                "operator":"equals",
                "field":"tags.app_name",
                "value":"global-search"
            },
            {
                "operator":"equals",
                "field":"tags.app_name",
                "value":"CaR"
            }
        ]
    }
}
```

## Variables
- URL_SENTRY - url for sentry
- SENTRY_AUTHORIZATION - sentry authorization key
- URL_MATOMO - url for matomo
- MATOMO_TOKEN - matomo token


The results what are added to the database are calculated from the number of sentry events and the number of visits retrieved from matomo.
We generate 2 measurements:
 - jsEvents with the values:
    - total_visits
    - sentry_events
    - value
 - serverErrors with the values:
    - total_visits
    - sentry_events
    - value

In both cases 
 - *total_visits* is read from matomo
 - *sentry_events* is the number of javascript/server errors read and filtered messages from sentry
 - *value* is sentry_events / total_visits * 100
