# Clone vs Transient Table: Storage Efficiency Breakdown
## Comparison
**Cloning with Copy-on-Write**
- No additional storage cost when cloning an existing table, clone shares same physical data as original
- When there are changes on clone, **copy-on-write** triggered and cost is charge on the changes on data

**Transient Table**
- Fully copy the original data (if selecting from other table)
- Storage costs for entire table

## Efficiency and Use Cases
**When Cloning is More Efficient**
- Small modifiation
  - Working on large original dataset and only need to add small amount of mock data
- Temporary operations
  - Testing without needing full duplicate of data
**When Transient Table is More Efficient**
- Ephemeral data
  - Need a temporary table for short-lived, intensive processes that don't require data retention or time travel
- Full data copy use case
  - If building completely separate data set that is expected to evolve independantly from original
