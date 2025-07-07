# ⚠ Limitations with ERD version 2 model
## Currently, the system works like this:
- Employee Submit vacation request  
- Only ONE Manager can approve


 ### ❌ This fails in real-world scenarios like:
 1. **single point of failure**
    - Only one manager can approve/reject what if the HR also needs to approve `(for payroll purposes)`
    - Request get stuck if manager is unavailable (`vacation`/`leave`)
 2. **No complex approval chains**
    - Multiple approvers `(Manager + HR)`
    - Esclation based on vacation duration 
    - Department head approval for long leaves 
      - `When long vacations need extra approval from a director`
 3. **WorkFlow Rules** 
    - Cannot configure different paths like :
    - Sequential approval (`Manager` → `HR`)
    - Parallel approval (Any 2 of 3 directors)
    - Conditional rules (`7+` days needs Director approval)
---




# ✅ Vacation Tracking System - ERD Version 3
## 🧠 Multi-Level Approval Workflow with Status Tracking
We'll transform it into a flexible levels system:

- Employee Submit vacation request  
- The Request will got through `level 1` (**Manager**)
- When the amanger approve it will go to `level 2`  (**HR**)
- Then it will go to  `level 3` (**Director**  `if needed`)
- And so on..

> [!Note]
> Some vacation requests needs simple approval from the `maanger` and `HR` only, and some requests needs approval from the `director` and maybe there are more levels between that .

##  Workflow examples
- `Basic Vacation`: Only needs `manager` approval
- `Extended Vacation`: Needs manager → `HR` → `director`


### 🌟 Real-World Example
> Scenario: Employee requests a 10-day vacation
> System Detects this needs the "Extended Vacation" workflow based : 

- level 1: Manager approves (Level 1 complete)
- level 2: HR verifies payroll impact (Level 2 complete)
- level 3: Director approves long absence (Final approval)

---

# 🧱Key Components For these requirements

## 📐✍  Workflow Template (ApprovalWorkflow)
Master workflow template which  different requests will follow based on  business rules
### Table structure
- Id (Primary Key)
- Name (e.g., "Standard Vacation")
- Description (Description for the workflow)

## ↗  Approval Level (ApprovalLevel)
Each workflow contains ordered levels which considerd as steps in the workflow:
### Table structure
- LevelId (Primary Key)
- WorkflowId (References ApprovalWorkflow `workflowId` )
- LevelNumber (Order for levels)
- ApprovalType (Enum Serial/Parallel)

## 🧑  Approver (Approver)
Who can approve at each level and each level can have multiple authorized people:

### Table structure
- LevelId (References ApprovalLevel `LevelId`)
- EmployeeId (References Employee `EmpId` )
- RoleId  (Optional)

## 📚 Request Approval history (RequestApproval)
Tracks each approval action and record it as ledger:

### Table structure
- RequestId (References Request `RequestId`)
- LevelId (References ApprovalLevel `LevelId` )
- ApproverId  (References Employee `EmpId`)
- Status  (Optional)
- Comment  (Optional)



| Table	           | Purpose                      | Key Fields                                                              |
|------------------|------------------------------|-------------------------------------------------------------------------|
|`ApprovalWorkflow`| Master workflow template     | `Id`, `Name`, `Description`                                             |
|`ApprovalLevel`   | Steps in workflow	          | `Id`, `WorkflowId`, `LevelNumber`, `ApprovalType` (SERIAL/PARALLEL)     |
|`Approver`	       | Who can approve at each level| `LevelId`, `EmployeeId`, `RoleId` (optional)                            |
|`RequestApproval` | Tracks each approval action  | `RequestId`, `LevelId`, `ApproverId`, `Status`, `Comments`, `ActionDate`|

---


# 🔄 Request Lifecycle
### Phase 1: Request Submission

 > Employee submits request:

```sql
INSERT INTO Request (Id, EmployeeId, StartDate, EndDate, Status, WorkflowId, CurrentLevel ) 
       VALUES  VALUES (1001, 501, '2024-09-01', '2024-09-05', 'Pending', 1, 1);
```

> System logs initial status in the `RequestStatusHistory`:

```sql
INSERT INTO RequestStatusHistory  ( RequestId, Status, ChangedById, Comment ) 
       VALUES (1001, 'Pending', 501, 'Initial submission');
```

### Database State After Submission
|Table                 |Field          |Value    |
|:--------------------:|:-------------:|:-------:|
|`Request`             |`Stuatus`      |`Pending`|
|`Request`             |`CurrentLevel` |`1`      |
|`RequestStatusHistory`|`Status`       |`Pending`|


### Phase 2: Routing to Approvers & Approval:

> System Routes to Approvers

```sql
SELECT e.Id, e.Name
FROM Approver a
JOIN Employee e ON a.EmployeeId = e.Id
WHERE a.LevelId = [CurrentLevel]
AND a.WorkflowId = [RequestWorkflowId];
```

### Approval Path Example:
1. Manager Approval (Level 1)
2. HR Verification (Level 2)
3. Director Approval `if >5 days` (Level 3)


&nbsp; 

> **Case A** : Serial Approval (Manager → HR)
> Manager approves approves at level 1 :

```sql
INSERT INTO RequestApproval (RequestId, LevelId, ApproverId, Decision, Comments )  
       VALUES (1001, 1, 601, 'Approved', NOW(), 'Coverage confirmed');
```

> System progresses to next level:

```sql
UPDATE Request SET CurrentLevel = 2 WHERE Id = 1001;
```

&nbsp; 

> **Case B** : Parallel Approval (Any HR Member)
> Any HR can approve without waiting:

```sql
INSERT INTO RequestApproval (RequestId, LevelId, ApproverId, Decision, Comments ) 
       VALUES (1001, 2, 701, 'Approved', NOW(), 'Payroll adjusted');
```

&nbsp; 

### Phase 3: Finalization

> Approval Completed:

```sql
UPDATE Request SET Status = 'Approved', CurrentLevel = NULL WHERE Id = 1001;
```

&nbsp; 

> Rejection Scenario:
> Any level can reject:

```sql
INSERT INTO RequestApproval VALUES (1001, 2, 701, 'Rejected', NOW(), 'Conflict with company event');
```

&nbsp; 

> System halts workflow:

```sql
UPDATE Request SET Status = 'Rejected', CurrentLevel = NULL WHERE Id = 1001;
```


### ⏱️ Escalation Handling
> Timeout Detection (Daily Job):

```sql
SELECT r.Id FROM Request r
JOIN ApprovalLevel l ON r.CurrentLevel = l.Id
WHERE r.Status = 'Pending'
AND l.IsEscalatable = 1
AND DATEDIFF(NOW(), r.LastUpdated) > 2;
```

**Escalation Actions:**
> Notify backup approver:

```sql
INSERT INTO Notifications VALUES (801, 'Escalated: Request 1001 pending >2 days', 1001);
```

> Skip to next level if configured:

```sql
UPDATE Request SET CurrentLevel = 3 WHERE Id = 1001;
```




