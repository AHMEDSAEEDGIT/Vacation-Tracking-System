# ✏ Use Case: Edit or withdraw pending request

This file describes how an employee edit or withdraw a pending vacation request before manager reviews it.

---

```pseudo
BEGIN
    DISPLAY login screen via Intranet Portal
    IF employee is authenticated THEN
        DISPLAY VTS Home Page with vacation summary

        PROMPT employee to select a pending request to edit
        DISPLAY editable request form (title, dates, comments)

        WAIT for employee input:
            - submit edited request OR
            - choose to withdraw

        IF employee submits edited request THEN
            VALIDATE input
            IF valid THEN
                UPDATE request
                RETURN to VTS Home Page
            ELSE
                REDISPLAY form with errors
            ENDIF
        ELSE IF employee chooses to withdraw THEN
            PROMPT for confirmation
            IF confirmed THEN
                WITHDRAW request
                RETURN to VTS Home Page
            ELSE
                CANCEL operation
            ENDIF
        ENDIF
    ELSE
        DISPLAY error: "Authentication Failed"
    ENDIF
END
```