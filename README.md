# Oracle APEX Resource Collection

This repository contains useful Oracle APEX Resources. If you want to add something new, feel free to fork this repo and send me a pull request.

## Table of Contents

[Creating Cron Jobs with DBMS_SCHEDULER](#cron-jobs) <br/>
[Creating Cards Using Classic Reports](#classic-report-cards)

## Creating Cron Jobs (Oracle APEX 19.2)

Cron jobs can be thought of as scheduled tasks, executed at regular time intervals, defined by the programmer. <br/> From the aforementioned description, one can see that they need to do two things. These include:

- define a task to be executed;
- define the intervals. <br/>

To accomplish the creating of cron jobs in Oracle APEX, the [DBMS_SCHEDULER](https://docs.oracle.com/database/121/ARPLS/d_sched.htm) can be used. The DBMS_SCHEDULER provides database driven jobs. <br/>

## Defining a task to be executed

Given that a cron job is expected to run the same query periodically, a personal recommendation would be to define the task in a [procedure](https://docs.oracle.com/database/apex-18.1/AEUTL/managing-procedures.htm#AEUTL149). Note, the task can also be defined in a PL/SQL Block.

```
CREATE [OR REPLACE] PROCEDURE procedure_name (param_name param_type, ...)
IS
    [declaration_section]

    BEGIN
        [executable_section]

    [EXCEPTION]
        [exception_section]
    END procedure_name;
```

## <a id="cron-jobs">Creating Cron Jobs with `DBMS_SCHEDULER`</a>

```
BEGIN
    DBMS_SCHEDULER.CREATE_JOB(
        job_name        => 'job_name',
        job_type        => 'STORED_PROCEDURE, PLSQL_BLOCK, EXECUTABLE',
        job_action      => 'procedure_name OR include PLSQL block',
        start_date      => SYSDATE.
        end_date        => SYSDATE,
        repeat_interval =>,
        enabled         => TRUE OR FALSE,
        comments        => 'job_comments'
    );
END;

```

### Viewing, Enabling/Disabling Schedules

The `ENABLE` method/function is used in the event that the 'enabled' property was set to `FALSE` when the schedule was created or if the `DISABLE` method/function was previously ran.

```
//VIEW ALL RUNNING SCHEDULES

SELECT job_name, job_action, start_date, end_date, repeat_interval, enabled, comments FROM SYS.ALL_SCHEDULER_JOBS

//ENABLE/DISABLE SCHEDULE

EXEC DBMS_SCHEDULER.ENABLE('job_name');
EXEC DBMS_SCHEDULER.DISABLE('job_name');
```

## Example

In the code snippet below, a procedure is first created and given the name `update_appointment_status`. After that is created, a scheduler job is created that executes the created procedure, every fifteen minutes.

```
CREATE OR REPLACE PROCEDURE update_appointment_status IS
    CURSOR c_appointments IS
    SELECT ID FROM appointments;

    BEGIN
    FOR appointment IN c_appointments LOOP
        UPDATE APPOINTMENTS
            SET STATUS = CASE
                WHEN
                    (
                        (SELECT TO_CHAR(SYSDATE - 4/24,'MM/DD/YYYY HH24:MI:SS') FROM DUAL) > TO_CHAR(END_TIME, 'MM/DD/YYYY HH24:MI:SS') AND
                        STATUS = 'Confirmed'
                    )
                    THEN 'Completed'
                    ELSE 'Cancelled'
                END
        WHERE ID=appointment.ID AND STATUS != 'Completed';
    END LOOP;
    END update_appointment_status;


BEGIN
    DBMS_SCHEDULER.CREATE_JOB(
        job_name        => 'update_appointment_status',
        job_type        => 'STORED_PROCEDURE',
        job_action      => 'UPDATE_APPOINTMENT_STATUS',
        start_date      => TO_DATE(TO_CHAR(SYSTIMESTAMP - 4/24, 'MM/DD/YYYY HH24:MI:SS') , 'MM/DD/YYYY HH24:MI:SS'),
        end_date        => NULL,
        repeat_interval => 'FREQ=MINUTELY; INTERVAL=15',
        enabled         => TRUE,
        auto_drop       => FALSE,
        comments        => 'This scheduled job automatically cancels expired appointments'
    );
END;

EXEC DBMS_SCHEDULER.DISABLE('update_appointment_status');

```

## <a id="classic-report-cards">Creating Cards Using Classic Reports</a>

- **CREATE**-is used when you are creating a new trigger
- **OR REPLACE**-is optional, and is used when you want oto modify an existing trigger
- **BEFORE|AFTER**-is used to specify when the trigger should be fired.
- **FOLLOWS**-is used when you have multiple triggers on the same table and you want to specify the order in which they will be fired.
- **[DISABLE/ENABLE]**-is used to set the status.

<br/><br/>

# Creating Cards Using Classic Reports

- Step 1: Add a region/subregion, with _Type_ **Classic Report**
- Step 2: Include the following Aliases: _CARD_TITLE, CARD_SUBTEXT, CARD_TEXT, CARD_LINK_.
- Step 3: Select _Attributes_ and set _Template_ to **Cards**.
- Step 4: Select _Template Options_ and set _Style_ to **Block**, _Layout_ to **Span Horizontally**, _Animation_ to **Raise Card**.

<br/>

- **CARD_TITLE** is the main title that is displayed at the first at the top of the card after the icon region.
- **CARD_SUBTEXT** comes immediately after the title.
- **CARD_TEXT** comes immediately after the subtext.
- **CARD_LINK** this is not visible. It is used to link a card to another resource upon a user event.

The SQL Query should look like the following:

```
SELECT
       'Total Goods' AS CARD_TITLE,
       '' AS CARD_SUBTEXT,
       COUNT(*) AS CARD_TEXT,
       NULL AS CARD_LINK
FROM GOODS
```

The code snippet above defines the card title as `Total Goods`. While the subtext is left blank, since there is no need for that. Following the subtext is the body of the card, the _card text_. The total `count` is used here. Lastly, no link was included because the card was not meant to redirect the user to some other resources.
