# Oracle APEX Resource Collection

This repository contains useful Oracle APEX Resources. If you want to add something new, feel free to fork this repo and send me a pull request.

## Table of Contents

[Creating Cron Jobs with DBMS_SCHEDULER](#cron-jobs)

## Creating Cron Jobs (Oracle APEX 19.2)

Cron jobs may be defined as scheduled tasks, executed on regular time intervals set by the programmer. <br/> From the definition, one can see that they need to do two things. These include:

- define a task to be executed (e.g. a procedure);
- define the intervals. <br/>

To accomplish the creating of cron jobs in Oracle APEX, the [DBMS_SCHEDULER](https://docs.oracle.com/database/121/ARPLS/d_sched.htm) will be used. The DBMS_SCHEDULER provides database driven jobs. <br/>

## Defining a task to be executed

Given that a cron job is expected to run the same query periodically, a personal recommendation would be to define the task in a [procedure](https://docs.oracle.com/database/apex-18.1/AEUTL/managing-procedures.htm#AEUTL149). Note, the task can also be defined in a PL/SQL Block.

```
CREATE [OR REPLACE] PROCEDURE procedure_name (param_name, param_type)
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
