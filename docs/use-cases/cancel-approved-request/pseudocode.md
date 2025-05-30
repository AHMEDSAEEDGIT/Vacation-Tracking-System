# ❌ Use Case: Cancel approved request.

This file describes how an employee cancels an approved  vacation request .

---

```pseudo
BEGIN
    DISPLAY login screen via Intranet Portal
    IF employee is authenticated THEN
        DISPLAY VTS home page with:
            - vacation request summary
            - time balance
            - status of active vacation requests (last 6 months to next 18 months)

        PROMPT employee to select a vacation time request to cancel
        IF selected request is approved AND (scheduled for future OR recent past) THEN
            IF request is in future THEN
                PROMPT for cancellation confirmation
            ELSE IF request is in recent past THEN
                PROMPT for confirmation AND explanation

            IF employee confirms THEN
                UPDATE request status to "Canceled"
                RESTORE vacation time to employee balance
                SEND email notification to manager
            ELSE
                ABORT cancellation
            ENDIF
        ELSE
            DISPLAY error: "Only approved future or recent past requests can be canceled"
        ENDIF

        RETURN to VTS home page
        REFRESH vacation summary
    ELSE
        DISPLAY error: "Authentication Failed"
    ENDIF
END
```