## Data Explorer
_(aka “ADX,” “Azure Data Explorer,” “Kusto”)_

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

### Frequently Asked Questions (FAQ)
The answers provided below represent best-known information on a given topic as of a given date.

#### Retention Policy
Recoverability … what happens if important data is deleted as a result of retention policy settings?
* September 26, 2022... Documentation explains that retention policy includes two properties:
  * SoftDeletePeriod, the time span for which data is guaranteed to be available to query (default: 100 years)
  * Recoverability, ensures that data will be recoverable for 14 days after soft delete (default: enabled)

#### Data Share
Cross-Region Cluster-Follower … when will it be possible to follow shared ADX A in region A from ADX B in region B?
* August 11, 2022... "no plans to re-implement this product feature... was previously implemented and then deprecated because it created unpredictable cost for customers"

Customer Managed Keys ... when will ADX Data Share support Customer Managed Keys?
* August 11, 2022... "product feature is planned, but no articulated ETA"

#### DevOps / GitHub
Is there any way to connect ADX directly to DevOps / GitHub (like Synapse **Set up code repository**)?
* August 11, 2022... "we do not have anything like that today outside Synapse"

#### Execute Multiple Commands at once
  ```
  .execute database script <|
   {command1};
   {command2};
  ```

#### Geo-Replication
Does ADX support geo-replication?
* August 11, 2022... "geo-replication is not an existing or planned product feature"

#### Kepler
Are there plans to bake **Kepler** into Azure Data Explorer?
* August 11, 2022... "no plans at the moment"

#### How do I figure out what table Column X is in?

```
.show database informix schema
| where ColumnName contains '{column_name}'
| project TableName,ColumnName
```
