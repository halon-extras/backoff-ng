# Backoff mode (NG)

HSL functions to enable backoff mode on sub-queues using dynamic queue policies based on responses from the [bounce-classifier](https://docs.halon.io/bounce-classifier/).

## Installation

Follow the [instructions](https://docs.halon.io/manual/comp_install.html#installation) in our manual to add our package repository and then run the below command.

### Ubuntu

```
apt-get install halon-extras-backoff-ng
```

### RHEL

```
yum install halon-extras-backoff-ng
```

## Configuration

The plugin must be. configured with an `address` and `port` that the [bounce-classifier](https://docs.halon.io/bounce-classifier/) is listening on.

### smtpd-app.yaml

```
plugins:
  - id: backoff-ng
    config:
      address: 127.0.0.1
      port: 8080
```

### smtpd-policy.yaml

```
policies:
  - fields:
      - localip
      - grouping
    conditions:
      - if:
          grouping: "&google"
        then:
          properties:
            backoff-concurrency: 2
            backoff-rate: 10/3600
            backoff-ttl: 3600
    default:
      properties:
        backoff-concurrency: 1
        backoff-rate: 5/3600
        backoff-ttl: 3600
  - fields:
      - tenantid
      - jobid
      - grouping
    default:
      properties:
        backoff-concurrency: 1
        backoff-rate: 5/3600
        backoff-ttl: 3600
  - fields:
      - tenantid
      - localip
      - grouping
    default:
      properties:
        backoff-concurrency: 1
        backoff-rate: 5/3600
        backoff-ttl: 3600
  - fields:
      - localip
      - remoteip
    default:
      properties:
        backoff-concurrency: 1
        backoff-rate: 5/3600
        backoff-ttl: 3600
```

**Properties**

- backoff-concurrency `number` - The `concurrency` that should be applied (for `concurrency` actions)
- backoff-rate `string` - The `rate` that should be applied (for `rate` actions)
- backoff-connectinterval `number` - The `connectinterval` that should be applied (for `connectinterval` actions)
- backoff-ttl `number` - How long the backoff should be enabled (for `concurrency`, `rate` and `connectinterval` actions)
- backoff-suspend `number` - How long the queue should be suspended (for `suspend` actions)

You can also add overrides for specific [classifications](https://docs.halon.io/bounce-classifier/api.html) like this:

```
backoff-graylisting-concurrency: 2
backoff-graylisting-rate: 10/3600
backoff-graylisting-ttl: 3600
```

## Exported functions

These functions needs to be [imported](https://docs.halon.io/hsl/structures.html#import) from the `extras://backoff` module path.

### enable_backoff(arguments, message)

**Params**

- arguments `array` - The [$arguments](https://docs.halon.io/hsl/postdelivery.html#v-z1) variable
- message `array` - The [$message](https://docs.halon.io/hsl/postdelivery.html#v-m1) variable

**Returns**

An array containing the policy and/or suspension that was applied (if any).
On error an exception will be thrown.

## Examples

### Example (Post-delivery)

```
import { enable_backoff } from "extras://backoff-ng";

if ($arguments["action"]) {
  // Failed deliveries
  $backoff = enable_backoff($arguments, $message);
}
```
