# Beckn Protocol Telemetry

## What is Telemetry?
Telemetry in Beckn Protocol is a monitoring system that helps track and collect data about how your application is performing on the network and can be used for creating qualitative metrics. Think of it as a health monitoring system that keeps track of all network interactions and helps you understand what's happening with your application.

## How to Configure Telemetry?
You can enable and configure telemetry in your `config.yaml` file:

```yaml
telemetry:
  enabled: true                   # Turn telemetry on/off
  url: "your-telemetry-url"      # Where to send telemetry data (complete endpoint)
  batchSize: 100                 # How many events/requests to collect before sending
  syncInterval: 30               # How often to send data (in minutes)
  redis_db: 3                    # Database to store telemetry data temporarily
```

## What Data is Collected?

### Network Communication Data
- Request and response details between BAP (buyer app) and BPP (seller platform)

### Transaction Data
- Transaction IDs
- Message IDs
- Beckn Actions performed (search, select, init, confirm, etc.)
- Status of transactions

### Context Information
- Source and target platform details
- Domain information
- Location data (when available)
- Timestamp of events

## How Does It Work?

1. Your protocol server generates events during normal operation
2. These events are collected and stored temporarily
3. Data is sent to your specified telemetry URL when either:
   - 100 events are collected (default batchSize)
   - 30 minutes have passed (default syncInterval in minutes)
4. If sending fails, data is safely stored and retried later

## Benefits of Using Telemetry

1. **Monitor Network Health**
   - See if your application is working properly
   - Track response times
   - Identify any connection issues

2. **Track Transactions**
   - Follow the flow of transactions
   - Ensure messages are being delivered
   - Monitor success rates

3. **Performance Insights**
   - Understand how your application performs
   - Identify areas for improvement
   - Track error rates

4. **Geographic Analysis**
   - Understand usage patterns by location
   - Monitor regional performance








