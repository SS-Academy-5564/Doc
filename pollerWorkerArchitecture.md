# Polling Strategy

For V1, the Poller Worker will use a fixed-delay polling strategy.

The worker runs at a configurable interval, for example every 60 seconds. During each iteration, it selects enabled monitors that are due for execution and processes them in limited batches.

A monitor is considered due when:

```sql
WHERE StatusId = 'Enabled'
  AND NextExecutionAt <= SYSUTCDATETIME()
```

## Monitor Scheduling and Selection

Each worker iteration performs the following steps:

1. Find enabled monitors where `NextExecutionAt` is less than or equal to the current UTC time.
2. Take a limited batch of due monitors.
3. Execute up to N monitor checks in parallel.
4. Persist a poll result for every completed execution.
5. Update the monitor’s NextExecutionAt.
6. Repeat after the configured worker interval.

For V1, `NextExecutionAt` is calculated after each execution:

```text
NextExecutionAt = execution completed time + PollingIntervalSeconds
```

This approach avoids overlapping executions for the same monitor within a single worker instance.

## Worker Implementation

The worker will be implemented using BackgroundService.

PollerWorker is responsible only for repeatedly triggering the polling process. The main polling logic should be placed in a separate service, such as PollingService.

```csharp
public sealed class PollerWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<PollerWorker> _logger;
    private readonly PollingWorkerOptions _options;

    public PollerWorker(
        IServiceScopeFactory scopeFactory,
        ILogger<PollerWorker> logger,
        IOptions<PollingWorkerOptions> options)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
        _options = options.Value;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        using var timer = new PeriodicTimer(
            TimeSpan.FromSeconds(_options.LoopIntervalSeconds));

        do
        {
            try
            {
                using var scope = _scopeFactory.CreateScope();
                var pollingService =
                    scope.ServiceProvider.GetRequiredService<IPollingService>();

                await pollingService.ProcessDueMonitorsAsync(stoppingToken);
            }
            catch (OperationCanceledException) when (
                stoppingToken.IsCancellationRequested)
            {
                break;
            }
            catch (Exception exception)
            {
                _logger.LogError(
                    exception,
                    "Poller Worker iteration failed.");
            }
        }
        while (await timer.WaitForNextTickAsync(stoppingToken));
    }
}
```

`PollerWorker` is a singleton hosted service.
A dependency injection scope is created for each iteration because `IPollingService` depends on commands/queries which are registered as scoped. Database connections are created and disposed separately by each data-access operation.

### Polling service
The polling service is responsible for:

1. Selecting due, enabled monitors.
2. Executing monitor checks.
3. Persisting poll result.
4. Handling and logging monitor-level errors. 

## HTTP Request Execution

HTTP requests should be executed through `IHttpClientFactory`.

Each monitor execution sends one HTTP request. The overall timeout is read from
the monitor's `PollingTimeoutSeconds` value.

Automatic redirects are disabled for V1 so that HTTP 3xx responses are returned
to the polling service and persisted as failures.

### Timeouts

For V1, `PollingTimeoutSeconds` defines an overall request timeout covering:

- DNS resolution
- Connection establishment
- TLS negotiation
- Waiting for the HTTP response
- Reading the response

Separate connection, response-header, and response-body timeouts are postponed
for future versions.

### Failures

| Situation                     | Stored result                     |
| ----------------------------- | --------------------------------- |
| HTTP 200–299                  | Success                           |
| HTTP 300–399                  | Failed with HTTP status code      |
| HTTP 400–499                  | Failed with HTTP status code      |
| HTTP 500–599                  | Failed with HTTP status code      |
| Request timeout               | Failed with `Timeout` status      |
| DNS, TLS, or connection error | Failed with `NetworkError` status |
| Worker shutdown cancellation  | No result is persisted            |

### Retries

Retries are intentionally excluded from V1.

Each scheduled monitor execution performs only one HTTP request. The next scheduled monitor execution acts as the next retry opportunity.

Retry policies, exponential backoff, retry handling for transient failures, and special handling for HTTP 429 or HTTP 503 responses may be added in future versions.

## Poll Result Persistence

Each monitor execution creates a separate poll result record.

The poll result record should contain:

- MonitorId
- CheckedAt
- IsSuccess
- StatusCode
- ResponseTimeMs
- Value
- RequestStatus
- ErrorMessage

RequestStatus may contain:

- Success
- Failed
- Timeout
- NetworkError
- UnexpectedError

The poll-result insert and monitor update must be committed atomically.
It means the changes are committed only after both operations succeed; otherwise, it is rolled back.

The monitor update:

- `CurrentValue` when a value is successfully extracted
- `LastCheckedAt`
- `NextExecutionAt`

If the transaction fails, neither change is committed and the monitor remains
due for a later worker iteration. Poll results must be persisted for both
successful and failed checks.

## Error Handling and Logging

Errors are handled at two levels: monitor-level and worker-level.

### Monitor-Level Errors

A failed monitor request must not interrupt processing of other monitors in the batch.

Expected failures, such as timeouts, DNS failures, TLS failures, connection failures, and non-success HTTP status codes, are persisted as failed poll results.

Unexpected exceptions are logged and persisted as UnexpectedError when possible.

### Worker-Level Errors

Unexpected errors during a worker iteration, such as database failures, are logged. The worker continues with the next scheduled iteration instead of stopping the host.

### Logs

Logs should include:

- Monitor identifier
- Execution duration
- HTTP status code
- Result status
- Error type
- Exception details for unexpected failures

Sensitive values such as authorization headers, credentials, request bodies, and response bodies must not be logged.

## Required Database Changes

### Monitors

Add:

- `NextExecutionAt DATETIME2 NOT NULL`

Assume these fields already exist:

- `PollingIntervalSeconds`
- `PollingTimeoutSeconds`
- `CurrentValue`
- `LastCheckedAt`
- `StatusId`

When a monitor is created or enabled, `NextExecutionAt` is set to the current
UTC time.

Add an index to support efficient selection of due monitors:

```text
(StatusId, NextExecutionAt)
```

### MonitorPollResults

Create a separate table for poll result history with:

- `Id`
- `MonitorId`
- `Value`
- `CheckedAt`
- `IsSuccess`
- `StatusCode`
- `ResponseTimeMs`
- `RequestStatus`
- `ErrorMessage`

Add:

- A foreign key from `MonitorId` to `Monitors(Id)`
- An index on `(MonitorId, CheckedAt DESC)`

## Required Configuration Changes

```json
{
  "PollingWorker": {
    "LoopIntervalSeconds": 60,
    "BatchSize": 50,
    "MaxConcurrentRequests": 10
  }
}
```

- `LoopIntervalSeconds` defines how often the worker checks for due monitors.
- `BatchSize` limits the number of monitors selected per iteration.
- `MaxConcurrentRequests` limits the number of parallel HTTP requests.

## V1 Limitations

- Only one Poller Worker instance is supported.
- Multiple instances may cause duplicate executions because there is no distributed lock or lease.
- No retries or exponential backoff.
- No priority scheduling.
- No poll result retention or cleanup.
- Redirects are always treated as failures; redirect behavior is not configurable.
- No per-monitor retry or backoff configuration.

## Future Improvements

- Support for multiple worker instances.
- Distributed locking or database leasing.
- Retry policies with exponential backoff.
- Configurable accepted HTTP status codes.
- Configurable redirect behavior.
- Per-monitor retry and backoff configuration.
- Priority scheduling.
- Poll result retention and cleanup.
- Metrics, health checks, and alerting.
- Adaptive polling intervals based on monitor state.
- Separate connection, response-header, and response-body timeouts.
