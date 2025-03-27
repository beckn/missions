# Beckn Protocol Telemetry

## What is Telemetry?
Telemetry in Beckn Protocol is a monitoring system that helps track and collect data about how your application is performing on the network and can be used for creating qualitative metrics. Think of it as a health monitoring system that keeps track of all network interactions and helps you understand what's happening with your application.

## Types of Telemetry
There are two types of telemetry that can be enabled in the protocol server:

1. **Network Telemetry**: Generates telemetry events as per the open network telemetry specifications
2. **Raw Telemetry**: Generates raw payload events

## How to Configure Telemetry?
Add the telemetry configuration in your `default-<bap/bpp>-client.yml` and `default-<bap/bpp>-network.yml` files:

```yaml
telemetry:
  network:
    url: ""              # URL for network telemetry platform
  raw:
    url: ""              # URL for raw telemetry platform
  batchSize: 100         # Events per batch
  syncInterval: 5        # Sync interval in minutes
  storageType: "LOCAL"   # Either LOCAL or REDIS
  backupFilePath: "backups"
  redis:
    db: 4
  messageProperties:     # Additional attributes to derive
    - key: "provider.name"
      path: "message.order.provider.descriptor.name"
    - key: "item.quantity" 
      path: "message.order.items[].quantity.selected.measure.value"
service:
  name: "network_service"    # Service name producing the event
  version: "1.0.0"          # Service version
```

### Configuration Properties

| Property | Description | Required | Default |
|----------|-------------|----------|---------|
| network.url | URL for sending telemetry to network data platform | Optional | - |
| raw.url | URL for sending raw telemetry to participant platform | Optional | - |
| batchSize | Number of telemetry events per batch | Yes | 100 |
| syncInterval | Time interval in minutes for sync | Yes | 5 |
| storageType | Storage type (LOCAL or REDIS) | Yes | LOCAL |
| backupFilePath | Path for backup telemetry data | Yes | backups |
| redis.db | Redis database index | If using REDIS | 4 |
| messageProperties | Additional attributes from context object | Optional | [] |
| service.name | Service name producing the event | Yes | - |
| service.version | Service version | Yes | - |

To disable telemetry, provide empty values for both network.url and raw.url.

## What Data is Collected?

### Network Communication Data
- Request and response details between BAP and BPP
- Additional custom attributes specified in messageProperties

### Transaction Data
- Transaction IDs
- Message IDs 
- Beckn Actions performed
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
   - Track application performance
   - Monitor response times
   - Identify connection issues

2. **Track Transactions**
   - Follow transaction flows
   - Ensure message delivery
   - Monitor success rates

3. **Performance Insights**
   - Analyze application performance
   - Identify improvement areas
   - Track error rates

4. **Geographic Analysis**
   - Understand usage patterns
   - Monitor regional performance








