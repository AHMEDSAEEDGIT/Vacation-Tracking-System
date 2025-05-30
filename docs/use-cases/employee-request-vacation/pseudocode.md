# 🧾 Use Case: Manage Time

This file describes how an employee submits a vacation request and how a manager reviews it.

---

## 🔐 Employee Logs In

```pseudo
BEGIN

// Employee logs into the system
IF authenticate(employeeCredentials) == TRUE THEN
    DISPLAY employeeDashboard()
    DISPLAY vacationHistory(6_months_past, 18_months_future)
ELSE
    DISPLAY "Login failed"
    EXIT
END IF

END
```

---

## 📝 Employee Submits Vacation Request
```pseudo
BEGIN

// Employee initiates a new request
IF userClicks("Create New Vacation Request") THEN
    categories = getVacationCategoriesWithPositiveBalance(employeeID)
    DISPLAY vacationRequestForm(categories)

    // Employee submits request
    INPUT dates, hoursPerDate, title, description

    // Validate the request
    IF validateRequest(dates, hoursPerDate, title, description) == FALSE THEN
        DISPLAY validationErrors()
        PROMPT userToEditOrCancel()
        EXIT
    END IF

    // Save the request
    requestID = saveVacationRequest(employeeID, ivacationRequest)

    // Check if approval is required
    IF requiresManagerApproval(employeeID) == TRUE THEN
        managerEmail = getManagerEmail(employeeID)
        SEND_EMAIL(to=managerEmail, subject="Vacation Request Approval Needed", linkToRequest(requestID))
        setRequestStatus(requestID, RequestStatus.PENDING_APPROVAL)
    ELSE
        setRequestStatus(requestID, RequestStatus.APPROVED)
        SEND_EMAIL(to=employeeEmail, subject="Vacation Request Approved")
    END IF

    DISPLAY "Request Submitted Successfully"
END IF

END
```

---

## ✅ Manager Reviews Request
```pseudo
BEGIN

// Manager approves/rejects request
IF managerLogsIn() == TRUE THEN
    pendingRequests = getPendingRequests(managerID)
    DISPLAY pendingRequests

    FOR EACH request IN pendingRequests DO
        DISPLAY requestDetails(request)
        INPUT managerDecision, optionalExplanation

        IF managerDecision == "Approve" THEN
            setRequestStatus(request.id, RequestStatus.APPROVED)
        ELSE IF managerDecision == "Reject" THEN
            setRequestStatus(request.id, RequestStatus.REJECTED)
            recordExplanation(request.id, optionalExplanation)
        END IF

        SEND_EMAIL(to=employeeEmail, subject="Vacation Request " + managerDecision)
    END FOR


END
```

