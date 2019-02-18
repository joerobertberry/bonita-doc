# Work execution audit

Sometimes it is useful to understand what is currently happening in the platform, in particular for work execution, see [Work service mechanism](execution-sequence-states-and-transactions.md).

This can be achieved using a specific logger called _Work execution audit_

## What can be tracked

### Abnormal work execution

The logger produces a _Warning_ each time a work takes too much time to be executed or it was _rescheduled_ too many times.

Example of warning log produced:
```
2019-02-11 14:54:14,541 WARN  [BONITA_WORK_AUDIT.EXECUTION] (Bonita-Worker-1-06) Potential abnormal execution detected - cause TOO_MUCH_EXECUTIONS. org.bonitasoft.engine.work.WorkDescriptor@3727bde7[uuid=0ca437df-cd1e-46f1-875a-a199cabbdf50,type=EXECUTE_FLOWNODE,tenantId=1,parameters={processDefinitionId=7836119026252730773, processInstanceId=2013, readyHumanTask=false, flowNodeInstanceId=80075},retryCount=5,executionThreshold=2019-02-11T13:54:14.539Z,executionCount=11,registrationDate=2019-02-11T13:53:43.224Z,abnormalExecutionDetected=true]
2019-02-11 14:54:14,846 WARN  [BONITA_WORK_AUDIT.EXECUTION] (Bonita-Worker-1-01) Potential abnormal execution detected - cause TOO_MUCH_EXECUTIONS. org.bonitasoft.engine.work.WorkDescriptor@4c18babb[uuid=208b32fe-c191-4c9b-9966-95cadc75e33a,type=EXECUTE_FLOWNODE,tenantId=1,parameters={processDefinitionId=7836119026252730773, processInstanceId=2015, readyHumanTask=false, flowNodeInstanceId=80087},retryCount=5,executionThreshold=2019-02-11T13:54:14.846Z,executionCount=9,registrationDate=2019-02-11T13:53:43.506ZabnormalExecutionDetected=true]
2019-02-11 14:54:15,084 WARN  [BONITA_WORK_AUDIT.EXECUTION] (Bonita-Worker-1-10) Potential abnormal execution detected - cause TOO_MUCH_EXECUTIONS. org.bonitasoft.engine.work.WorkDescriptor@257ed754[uuid=eb1e719d-1baa-42dc-9ddc-bfdadd7e0ebe,type=EXECUTE_FLOWNODE,tenantId=1,parameters={processDefinitionId=7836119026252730773, processInstanceId=2017, readyHumanTask=false, flowNodeInstanceId=80103},retryCount=5,executionThreshold=2019-02-11T13:54:15.084Z,executionCount=10,registrationDate=2019-02-11T13:53:43.808Z,abnormalExecutionDetected=true
```

We can wee:

* the type of issue: `TOO_MANY_EXECUTIONS` OR `TOO_MUCH_TIME_ELAPSED_SINCE_REGISTRATION`
* information on the work:
  * uuid
  * type
  * parameters
  * executionCount: number of times work was rescheduled
  * registrationDate: date at which the work was registered
  * abnormalExecutionDetected: if we detected that there might be execution issue on that work

For theses cases it also produces an _Info_ log when the work was finally executed.

A _reschedule_ happens when a work can't be executed right now because some other work already locked the same process instance.

## How to activate it

### Logging configuration

Enable the logger called `BONITA_WORK_AUDIT.EXECUTION` in the logger configuration.


#### Example configuration ()

##### Tomcat `logging.properties`
```properties
# Specific logger for auditing work execution
BONITA_WORK_AUDIT.EXECUTION.level = INFO 
```

##### Wildfly `standalone.xml`
```xml
            <!-- Specific logger for auditing work execution -->
            <logger category="BONITA_WORK_AUDIT.EXECUTION">
                <level name="INFO" />
            </logger>
```

### Activate and configure the work execution audit

In `bonita-tenant-community-custom.properties` set flag `bonita.tenant.work.audit.activated` to `true`


Some properties can be configured

* `bonita.tenant.work.audit.abnormal.execution.threshold.execution_count`: will trigger a warning if this amount of reschedule is reached
* `bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_amount`: will not trigger the warning if the work age is less than this value
* `bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_unit`: unit of time in ChronoUnit (SECONDS, MINUTES, HOURS, ...)
* `bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_amount`: will trigger a warning if the work age is greater than this amount
* `bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_unit`: unit of time in ChronoUnit (SECONDS, MINUTES, HOURS, ...)

Example configuration:

```properties
# Work execution audit
# Durations must be ChronoUnit enum values
bonita.tenant.work.audit.activated=true
# Too many executions threshold
bonita.tenant.work.audit.abnormal.execution.threshold.execution_count=10
bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_amount=10
bonita.tenant.work.audit.abnormal.execution.threshold.execution_count_duration_unit=MINUTES
# Too much time elapsed since registration
bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_unit=MINUTES
bonita.tenant.work.audit.abnormal.execution.threshold.elapsed_duration_since_registration_amount=30
```
