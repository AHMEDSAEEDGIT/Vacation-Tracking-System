[ERD V2](VTS-ERD-V2%20(2).png)

# Vacation Tracking System (VTS) - ERD Version 

## Problem in V1
We can't track when status changes occurred or who made them, which is crucial for reporting and compliance.

&nbsp; 

### 🔑 Solution 
Create a new `RequestStatusHistory` table with these fields:
- `Id` (PK)
- `RequestId` (FK to Request)
- `Status` (enum: Pending/Approved/Rejected)
- `ChangedById` (FK to Employee - manager made the change)
- `ChangedAt` (timestamp)
- `Comment` (optional justification)

&nbsp;

### 🛠 Changes to Existing Structure:
1. Keep the `Status` field in `Request` table for current state for simplicty
2. Remove any approval-related fields from `Request` table

## 💡 Implementation Logic:
1. When status changes:
  - Insert new record in RequestStatusHistory
  - Update Status in Request table
2. To get full history:
   ```sql 
   SELECT * FROM RequestStatusHistory 
   WHERE RequestId = :requestId
   ORDER BY MangedAt DESC
   ```
3. ✒ Add Approval Comments

> [!NOTE]
> Problem in `V1`:
> No way to capture why a request was approved/rejected.

> 🔑 Solution:
> Use the Comment field in RequestStatusHistory table added above.

--- 

## ✅ New Approval Flow:
1. Employee submits request → Creates first history record (Pending)
2. Manager approves/rejects → Creates second history record
3. System can now:
  - Show full audit trail
  - Calculate approval times
  - Track multiple status changes

&nbsp;

### Get request with full history
```sql
SELECT r.*, h.Status, h.ManagedAt, e.Name AS e.Name ManagerName
FROM Request r
JOIN RequestStatusHistory h ON r.Id = h.RequestId
JOIN Employee e ON h.ManagedBy = e.Id
WHERE r.Id = :requestId
ORDER BY h.ManagedAt DESC
```
&nbsp; 

### 📝 Calculate time in pending state:

```sql
SELECT TIMESTAMPDIFF(HOUR, MIN(ManagedAt), MAX(ManagedAt))
FROM RequestStatusHistory
WHERE RequestId = :requestId AND Status = 'Pending'
```

---

## 📈 Version Comparison: V1 vs V2

| Feature/Capability       | V1                          | V2                          |
|--------------------------|-----------------------------|-----------------------------|
| **Implementation**       | Simpler to implement        | More complex due to history |
| **Database Structure**   | Fewer tables                | Additional history table    |
| **Query Performance**    | Faster for basic queries    | Slightly slower for history |
| **Audit Trail**          | ❌ No history tracking      | ✅ Full audit trail         |
| **Response Metrics**     | ❌ No timing data           | ✅ Response time metrics   |
| **Status Flexibility**   | ❌ Single status            | ✅ Multiple status changes  |
| **Comments**            | ❌ No justification system  | ✅ Comment field available  |
| **Compliance**          | ⚠️ Basic tracking only     | ✅ Better compliance        |
| **Approval Workflow**   | ❌ Single-step              | ⚠️ Still single-approver   |
| **Delegation**          | ❌ No support               | ❌ Still no support        |

**Key Improvements in V2:**
- Added comprehensive audit logging
- Enabled status change tracking
- Introduced comment/justification system
- Gained ability to measure response times

**Remaining Limitations (for V3):**
- Still single-approver workflow
- No delegation capabilities
- No SLA/timeout handling