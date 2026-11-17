# Food Delivery – Orders API and Architecture Decisions

Practical class: *Write an ADR & Prepare an OpenAPI Draft* · Variant 3 (Food-delivery platform)

**Team VetCare:** Ma Yitong · Pan Yaqin · Жанарбек Серлан · Бақытбай Бұлдірген

## What is here

| Path | Content |
|---|---|
| `docs/adr/README.md` | ADR index (number, title, status) |
| `docs/adr/0001-record-architecture-decisions.md` | Why we use ADRs (MADR template) |
| `docs/adr/0002-use-rabbitmq-between-orders-and-delivery.md` | Decision: RabbitMQ between Orders and Delivery |
| `api/openapi.yaml` | OpenAPI 3.1 draft of the Orders API (orders, order status) |
| `.spectral.yaml` | Lint rules: Spectral OpenAPI rules + 2 team rules |

## How to view the docs

- **ADRs:** open the Markdown files in `docs/adr/` on GitHub.
- **API in the browser:** open https://editor.swagger.io, choose *File → Import file* and select `api/openapi.yaml`.
- **Static HTML (Redoc):** `npx @redocly/cli build-docs api/openapi.yaml -o docs/api.html`

## Validate and mock

```bash
npx @stoplight/spectral-cli lint api/openapi.yaml          # uses .spectral.yaml
npx @stoplight/prism-cli mock api/openapi.yaml             # mock server on http://127.0.0.1:4010
curl -H "Authorization: Bearer test" http://127.0.0.1:4010/orders/1b4e28ba-2fa1-11d2-883f-0016d3cca427
```
