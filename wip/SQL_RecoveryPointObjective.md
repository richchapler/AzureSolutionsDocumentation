# SQL Server, Recovery Point Objective (RPO)

## Sources

- **[Monitor performance for Always On availability groups >> Estimate data loss (RPO)](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/monitor-performance-for-always-on-availability-groups?view=sql-server-ver16#:~:text=The%20log%20send%20queue%20represents%20all%20the%20data%20that%20can%20be%20lost%20from%20a%20catastrophic%20failure)**  
  Explains how RPO is measured and visualized in Always On AGs:
  - **Log Send Queue**: Represents unreplicated log; potential data loss in failover.
  - **Commit Timestamp Gap**: Difference between primary and secondary commit times indicates RPO in seconds.
  - **SSMS Dashboard**: Displays estimated data loss (RPO) per replica.

- **[Troubleshoot: Potential data loss with asynchronous-commit availability-group replicas](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/troubleshoot-availability-group-exceeded-rpo?view=sql-server-ver16#:~:text=A%20synchronous-commit%20secondary%20replica%20guarantees%20zero%20data%20loss%2C%20but%20the%20potential%20data%20loss%20of%20an%20asynchronous-commit%20secondary%20replica%20depends%20on%20how%20much%20log%20is%20still%20waiting%20to%20be%20hardened%20on%20the%20secondary%20replica)**  
  Details common reasons for RPO exceedance, especially in asynchronous mode:
  - **Async Replica Risk**: Data loss depends on unsent or un-hardened log.
  - **Bottlenecks**: Slow disk I/O, network latency, or limited throughput can inflate the log send queue.
  - **Monitoring Guidance**: Log send rate vs. flush rate, flow control wait times.

- **[Failover and Failover Modes (Always On Availability Groups)](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/failover-and-failover-modes-always-on-availability-groups?view=sql-server-ver16#:~:text=automatic%20failover%20(without%20data%20loss)%2C%20planned%20manual%20failover%20(without%20data%20loss)%2C%20and%20forced%20manual%20failover%20(with%20possible%20data%20loss))**  
  Clarifies how replication and failover modes impact RPO:
  - **Synchronous Commit**: Enables automatic/planned failover with no data loss (RPO = 0).
  - **Asynchronous Commit**: Requires forced failover with potential data loss.
  - **Availability Mode Matrix**: Highlights failover type compatibility per replica mode.

- **[What is an Always On availability group?](https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server?view=sql-server-ver16#:~:text=Secondary%20databases%20aren%27t%20backups.%20Continue%20to%20back%20up%20your%20databases%20and%20their%20transaction%20logs%20regularly)**  
  Emphasizes backup importance in RPO planning:
  - **AG Replicas ≠ Backups**: Replication does not replace backup strategy.
  - **Backup Frequency**: Critical to achieve RPO targets for site-wide disasters.
  - **Catastrophic Recovery**: Backups define true RPO if all replicas are lost.

-------------------------

## Recovery Point Objective (RPO): How Much Data Is Lost

**Recovery Point Objective (RPO)** defines the maximum amount of data that can be lost in a failure scenario. In SQL Server Always On availability groups, RPO is directly tied to how much transaction log has not been replicated from the primary to secondary replicas at any given moment. It is a measurement of exposure—the window of time or volume of data that would be lost if the primary fails right now.

### Measuring RPO

RPO is typically measured in seconds and derived from two key metrics:

- **Commit Timestamp Lag**: The time difference between the most recent committed transaction on the primary and the most recent hardened transaction on a secondary.
- **Log Send Queue Size**: The volume of transaction log data (in KB or MB) sitting on the primary that has not yet been sent to or hardened on the secondary.

You can get an accurate, real-time RPO value using:

- **SSMS Availability Groups Dashboard**: Add the column "Estimated Data Loss (time)" to see the current RPO for each secondary.
- **`sys.dm_hadr_database_replica_states` DMV**: Compare `last_commit_time` values between primary and secondary replicas.

Example calculation:
```sql
SELECT
    drs.database_id,
    drs.replica_id,
    drs.last_commit_time,
    drs.redo_queue_size,
    drs.log_send_queue_size
FROM sys.dm_hadr_database_replica_states drs;
```

This tells you the last commit time for each replica and the size of the unsent log queue.

### What Causes RPO to Increase?

Even in well-configured environments, asynchronous replicas will always have some non-zero RPO. Contributing factors include:

- **Network latency and bandwidth limitations**: If the primary generates log faster than the network can deliver, the queue grows.
- **Slow I/O on the secondary**: Log cannot be hardened fast enough, so acknowledgments are delayed.
- **Workload spikes**: Sudden bursts of transactions on the primary create replication backlogs.
- **Flow control**: SQL Server throttles replication when the secondary can't keep up.

In synchronous commit mode, RPO should be zero—but only if the replica is healthy and in a `SYNCHRONIZED` state. If not, it behaves like an async replica.

### How to Minimize RPO

- **Use synchronous commit for HA replicas**
- **Keep secondaries in `SYNCHRONIZED` state**
- **Ensure fast disk I/O on secondaries**
- **Use high-bandwidth, low-latency network paths**
- **Monitor `log_send_queue_size` and `last_commit_time` gap continuously**
- **Set alerts on estimated data loss exceeding thresholds**
- **Test failover scenarios and measure actual loss**
- **Back up frequently—even with AGs, backups are your last line of defense**

### Interpreting RPO in Practice

If your RPO SLA is 30 seconds and the estimated data loss on your async replica regularly exceeds 2 minutes, you are out of compliance—even if no failover has occurred yet. This requires immediate action (e.g., investigate network throughput, disk latency, or reduce load).

If using only async replicas, define your risk tolerance clearly—"We can afford to lose up to 1 minute of data"—and use monitoring to enforce it. If not, use synchronous replicas.