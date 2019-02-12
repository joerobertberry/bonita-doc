# Work execution audit

Sometimes it is useful to understand what is currently happening in the platform, in particular for work execution, see [Work service mechanism](execution-sequence-states-and-transactions.md).

This can be achieved using a specific logger called _Work execution audit_


## How to activate it

### Enable the logger

Enable the logger called `BONITA_WORK_AUDIT.EXECUTION` in the logger configuration.


### Activate and configure the work execution audit

In `bonita-tenant-community-custom.properties` set flag `bonita.tenant.work.audit.activated` to `true`


Some properties can be configured

* `bonita.tenant.work.audit.abnormal.execution.threshold.execution_count`: will trigger a warning if this amount of reschedule is reached
* `bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_amount`: will not trigger the warning age of work is less than this value
* `bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_unit`: unit of time in ChronoUnit (SECONDS, MINUTES, HOURS, ...)
* `bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_amount`: will trigger a warning if the work age is greater than this amount
* `bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_unit`: unit of time in ChronoUnit (SECONDS, MINUTES, HOURS, ...)

Example configuration:

```properties
# Work execution audit
# Durations must be ChronoUnit enum values
bonita.tenant.work.audit.activated=true
# Too much executions threshold
bonita.tenant.work.audit.abnormal.execution.threshold.execution_count=10
bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_amount=10
bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_unit=MINUTES
# Too much time elapsed since registration
bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_unit=MINUTES
bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_amount=30
```


## What can be tracked

### Abnormal work execution

The logger produce a _Warning_ each time a work takes too much time to be executed or it was _rescheduled_ too much times.

Example of warning log produced:
```
2019-02-11 14:54:14,541 WARN  [BONITA_WORK_AUDIT.EXECUTION] (Bonita-Worker-1-06) Potential abnormal execution detected - cause TOO_MUCH_EXECUTIONS. org.bonitasoft.engine.work.WorkDescriptor@3727bde7[uuid=0ca437df-cd1e-46f1-875a-a199cabbdf50,type=EXECUTE_FLOWNODE,tenantId=1,parameters={processDefinitionId=7836119026252730773, processInstanceId=2013, readyHumanTask=false, flowNodeInstanceId=80075},retryCount=5,executionThreshold=2019-02-11T13:54:14.539Z,executionCount=11,registrationDate=2019-02-11T13:53:43.224Z]
2019-02-11 14:54:14,846 WARN  [BONITA_WORK_AUDIT.EXECUTION] (Bonita-Worker-1-01) Potential abnormal execution detected - cause TOO_MUCH_EXECUTIONS. org.bonitasoft.engine.work.WorkDescriptor@4c18babb[uuid=208b32fe-c191-4c9b-9966-95cadc75e33a,type=EXECUTE_FLOWNODE,tenantId=1,parameters={processDefinitionId=7836119026252730773, processInstanceId=2015, readyHumanTask=false, flowNodeInstanceId=80087},retryCount=5,executionThreshold=2019-02-11T13:54:14.846Z,executionCount=9,registrationDate=2019-02-11T13:53:43.506Z]
2019-02-11 14:54:15,084 WARN  [BONITA_WORK_AUDIT.EXECUTION] (Bonita-Worker-1-10) Potential abnormal execution detected - cause TOO_MUCH_EXECUTIONS. org.bonitasoft.engine.work.WorkDescriptor@257ed754[uuid=eb1e719d-1baa-42dc-9ddc-bfdadd7e0ebe,type=EXECUTE_FLOWNODE,tenantId=1,parameters={processDefinitionId=7836119026252730773, processInstanceId=2017, readyHumanTask=false, flowNodeInstanceId=80103},retryCount=5,executionThreshold=2019-02-11T13:54:15.084Z,executionCount=10,registrationDate=2019-02-11T13:53:43.808Z
```


For theses cases it also produce an _Info_ log when the work was finally executed.


A _reschedule_ happens when a work can't be executed right now because some other work already locked the same process instance.
