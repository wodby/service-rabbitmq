# RabbitMQ on Wodby

What Wodby sets up for this RabbitMQ service.

## How applications reach it

- Host: the name of this app service inside the environment.
- Ports: `5672` AMQP (no TLS), `15672` management UI and HTTP API (the service's main HTTP endpoint), `15692` Prometheus metrics at `/metrics` (private, internal only).
- Credentials are tokens of this service: `username` (a fixed account name), `password` (generated) and `vhost` (`/`). A service linked to this one receives host, port and these tokens in variables named by the linking service; read them in code and do not copy the password into the repository.
- Inside this container the same values are in `RABBITMQ_DEFAULT_USER`, `RABBITMQ_DEFAULT_PASS` and `RABBITMQ_DEFAULT_VHOST`. The same account signs in to the management UI.
- An Erlang cookie is also generated (token `erlang_cookie`); applications do not need it.

## What is configured

- The plugins `rabbitmq_management` and `rabbitmq_prometheus` are enabled.
- On every start the container writes `/etc/rabbitmq/conf.d/90-wodby.conf` from environment variables. Never edit it; set the variables on the service:

| Variable | Setting | Default |
| --- | --- | --- |
| `RABBITMQ_CHANNEL_MAX` | `channel_max` | `2047` |
| `RABBITMQ_HEARTBEAT` | `heartbeat` | `60` |
| `RABBITMQ_VM_MEMORY_HIGH_WATERMARK_RELATIVE` | memory high watermark, share of available memory | `0.4` |
| `RABBITMQ_DISK_FREE_LIMIT` | `disk_free_limit.absolute` | `50MB` |

- The default user, password and virtual host are applied by RabbitMQ only when it first creates its database. Later changes to the account are made in RabbitMQ itself (`rabbitmqctl`, management UI), not by changing the variables.
- The service runs as a single node and is not scalable.

## Data

Broker data (definitions, queues, messages) is in `/var/lib/rabbitmq`. The `data` volume is optional; without it everything, including users and queues created at runtime, is lost when the container is replaced. A message survives a restart only when the volume exists and the queue is durable and the message persistent. The manifest declares no backups, imports or actions.

## Check the result

From this service's container:

- `rabbitmq-diagnostics -q ping` and `rabbitmq-diagnostics -q status`
- `rabbitmqctl list_queues name messages consumers`
- `rabbitmqctl list_users` and `rabbitmqctl list_permissions -p /`
