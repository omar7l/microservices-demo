# SDN Grafana Certification Fork Notes

This fork is used for the SDN Grafana Partner L1 demo.

Base repo: https://github.com/GoogleCloudPlatform/microservices-demo
Fork: https://github.com/omar7l/microservices-demo
Branch: `sdn-grafana-otel-demo`

## Why fork

The first demo path used official Online Boutique images plus Grafana Beyla zero-code instrumentation.

That is useful, but Omar correctly challenged that real-world observability often needs SDK/manual OpenTelemetry for critical business flows.

This fork adds manual business-level spans to `checkoutservice` so the demo can compare:

```text
Direct/backend pipelines:
  Kubernetes metrics -> Alloy -> Grafana Cloud Mimir
  Pod logs           -> Alloy -> Grafana Cloud Loki

OpenTelemetry app instrumentation:
  checkoutservice SDK spans -> OTLP -> Alloy -> Grafana Cloud Tempo
```

## Modified service

`src/checkoutservice/main.go`

Added child spans around the checkout business flow:

```text
checkout.place_order
checkout.prepare_cart_and_shipping
checkout.calculate_order_total
checkout.charge_payment
checkout.ship_order
checkout.empty_cart
checkout.send_confirmation_email
```

Added business attributes such as:

```text
app.business_flow=checkout
app.user_currency
app.shipping_country
app.order_id
app.cart.item_count
app.order.item_count
app.order.currency
app.order.total_units
app.payment.amount_units
app.payment.transaction_id
app.shipping.tracking_id
app.checkout.result
```

## Why this matters

Beyla can show baseline request/service telemetry without code changes.

Manual/SDK OpenTelemetry can show semantic business steps and attributes that Beyla cannot infer from the outside.

This is the certification explanation:

> For the whole cluster, Alloy collects Kubernetes metrics and pod logs directly into Mimir and Loki. For the critical checkout flow, I use SDK-based OpenTelemetry instrumentation so Tempo shows business-level spans like charge payment and ship order. Beyla is useful for quick zero-code onboarding, but SDK instrumentation is stronger for high-value production flows.

## Runtime note

The custom fork image must be built and deployed for the new spans to appear. If the cluster still uses Google's official checkoutservice image, the forked code is not running yet.
