# ADR-0002: Use RabbitMQ for communication between Orders and Delivery

- Status: accepted
- Deciders: Ma Yitong, Pan Yaqin, Жанарбек Серлан, Бақытбай Бұлдірген
- Date: 2026-11-17

## Context and Problem Statement

Our food-delivery platform has an **Orders** service (customers place and track orders) and a
**Delivery** service (assigns couriers and reports pick-up and drop-off). When an order is placed,
Delivery must learn about it, and Orders must learn every status change from Delivery.
**Which communication style should connect the Orders and Delivery services?**

Assumptions (estimates):
- One city, about 2,000 orders per day; normal peak 50 orders/minute; promotions up to 500 orders/minute.
- Team of 4 students who know Python and HTTP; no dedicated operations engineer.
- Cloud deployment; budget for messaging infrastructure at most USD 100 per month.
- Delivery is a separate service and may be restarted or unavailable for several minutes during deployments.

## Decision Drivers

- D1 Reliability: no order is lost if Delivery is unavailable for up to 30 minutes.
- D2 Customer latency: `POST /orders` answers in p95 ≤ 300 ms, independent of Delivery.
- D3 Scalability: absorb a 10× peak (500 orders/minute) without failed orders.
- D4 Operational complexity and team skills: at most one extra managed component; the team can run it without a dedicated ops engineer.
- D5 Cost: messaging infrastructure ≤ USD 100 per month.
- D6 Status freshness: a status change in Delivery is visible to the customer within 5 seconds.

## Considered Options

1. Synchronous REST calls between Orders and Delivery (current prototype)
2. Message broker RabbitMQ (managed), with a queue per consumer
3. Event-streaming platform Apache Kafka (managed)

## Decision Outcome

Chosen option: **"Message broker RabbitMQ"**, because it is the only option that meets D1 and D2
together while staying within D4 and D5. REST fails D1 and D2 when Delivery is slow or down.
Kafka also meets D1–D3, but its operational model (partitions, consumer groups, offsets) and cost are
more than our load needs.

We will publish an `order.placed` message from Orders and `delivery.status_changed` messages from
Delivery, using persistent messages, publisher confirms and manual consumer acknowledgements.

### Consequences

- Good, because Orders accepts an order even when Delivery is down; messages wait in the queue (D1, D2).
- Good, because peaks are buffered in the queue and Delivery processes them at its own speed (D3).
- Bad, because the system becomes eventually consistent: right after `POST /orders` the order has
  status `placed` and no courier yet. Mitigation: the API returns the status explicitly and the
  customer app polls `GET /orders/{orderId}`; the status history shows each change with a timestamp.
- Bad, because a message can be delivered twice (at-least-once delivery). Mitigation: every message
  carries an `eventId`; consumers store processed IDs and ignore duplicates.
- Bad, because saving an order and publishing a message are two steps and one can fail. Mitigation:
  transactional outbox pattern – Orders writes the message to an outbox table in the same database
  transaction and a relay publishes it.
- Neutral: the team must learn basic RabbitMQ concepts (exchange, queue, acknowledgement); estimated 1–2 days.

### Confirmation

- An architecture test in CI fails if the Orders code imports an HTTP client for the Delivery service.
- Monitoring: queue length < 1,000 messages and status delay (D6) p95 < 5 s, checked weekly.
- Load test before release: 500 orders/minute for 10 minutes with Delivery stopped for 5 minutes; zero lost orders.
- Review date: 2027-05-01, or earlier if load grows above 5,000 orders/minute.

## Pros and Cons of the Options

| Driver | REST (sync) | RabbitMQ | Kafka |
|---|---|---|---|
| D1 Reliability when Delivery is down | −− | ++ | ++ |
| D2 Latency of `POST /orders` | − | ++ | ++ |
| D3 10× peak | − | + | ++ |
| D4 Operational complexity / skills | ++ | + | −− |
| D5 Cost ≤ USD 100/month | ++ | + | − |
| D6 Status within 5 s | ++ | + | + |

### Synchronous REST

- Good, because the team already knows HTTP and it needs no new component (D4, D5).
- Good, because a status change is visible immediately (D6).
- Bad, because if Delivery is down or slow, `POST /orders` fails or waits; retries must be built by hand (D1, D2) [1].
- Bad, because a 10× peak hits Delivery directly with no buffer (D3).

### RabbitMQ

- Good, because persistent messages, publisher confirms and consumer acknowledgements keep messages until they are processed [2].
- Good, because queues absorb peaks; our peak (≈ 8 messages/second) is far below what a single broker handles.
- Good, because managed RabbitMQ is offered by several providers with small plans [3].
- Bad, because delivery is at-least-once; consumers must be idempotent [2].

### Kafka

- Good, because it stores events durably and lets new consumers replay history [4].
- Good, because it scales to very high throughput through partitions (D3).
- Bad, because production clusters normally use replication across at least three brokers, and the team
  must learn partitions, consumer groups and offsets (D4) [4].
- Bad, because managed Kafka pricing is usage- and cluster-based and is aimed at much higher volumes than ours (D5) [5].

## More Information

1. Microservices.io – Pattern: Remote procedure invocation: https://microservices.io/patterns/communication-style/rpi.html
2. RabbitMQ documentation – Reliability guide: https://www.rabbitmq.com/docs/reliability and Consumer Acknowledgements and Publisher Confirms: https://www.rabbitmq.com/docs/confirms
3. CloudAMQP – Plans (managed RabbitMQ): https://www.cloudamqp.com/pricing
4. Apache Kafka documentation: https://kafka.apache.org/documentation/
5. Confluent Cloud pricing: https://www.confluent.io/confluent-cloud/pricing/
6. Microservices.io – Pattern: Transactional outbox: https://microservices.io/patterns/data/transactional-outbox.html
- Related: the API in `api/openapi.yaml` follows this decision (see its `info.description`).
