# Monitor Pattern

This tutorial demonstrates how to run a workflow in a loop. This can be used for recurring tasks that need to be executed on a certain frequency (e.g. a clean-up job that runs every hour). For more information on the monitor pattern see the [Dapr docs](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-patterns/#monitor).

## Inspect the code

Open the `MonitorWorkflow.java` file in the `tutorials/workflow/java/monitor-pattern/src/main/java/io/dapr/springboot/examples/monitor/` folder. 
This file contains the definition for the workflow that calls the `CheckStatus` activity to checks to see if a fictional resource is ready.  The `CheckStatus` activity uses a random number generator to simulate the status of the resource. If the status is not ready, the workflow will wait for one second and is continued as a new instance.

```mermaid
graph LR
   SW((Start
   Workflow))
   CHECK[CheckStatus]
   IF{Is Ready}
   TIMER[Wait for a period of time]
   NEW[Continue as a new instance]
   EW((End
   Workflow))
   SW --> CHECK
   CHECK --> IF
   IF -->|Yes| EW
   IF -->|No| TIMER
   TIMER --> NEW
   NEW --> SW
```

## Run the tutorial

1. Use a terminal to navigate to the `tutorials/workflow/java/monitor-pattern` folder.
2. Build and run the project using Maven.

    ```bash
    mvn spring-boot:test-run
    ```

3. Use the POST request in the [`monitor.http`](./monitor.http) file to start the workflow, or use this cURL command:

    ```bash
    curl -i --request POST http://localhost:8080/start/0
    ```

    The input for the workflow is an integer with the value `0`.

    The expected app logs are as follows:

    ```text
    [monitor-pattern]  io.dapr.springboot.examples.monitor.CheckStatusActivity : Received input: 0
    [monitor-pattern]  io.dapr.springboot.examples.monitor.CheckStatusActivity : Received input: 1
    [monitor-pattern]  io.dapr.springboot.examples.monitor.CheckStatusActivity : Received input: 2
    [monitor-pattern]  io.dapr.springboot.examples.monitor.CheckStatusActivity : Received input: 3
    [monitor-pattern]  io.dapr.springboot.examples.monitor.CheckStatusActivity : Received input: 4
    [monitor-pattern]  io.dapr.springboot.examples.monitor.CheckStatusActivity : Received input: 5

    ```

    *Note that the number of app log statements can vary due to the randomization in the `CheckStatus` activity.*

4. Use the GET request in the [`monitor.http`](./monitor.http) file to get the status of the workflow, or use this cURL command:

    ```bash
    curl --request GET --url http://localhost:8080/output
    ```

    The expected serialized output of the workflow is:

    ```txt
    "\"Status is healthy after checking 5 times.\""
    ```

    *The actual number of checks can vary since some randomization is used in the `CheckStatus` activity.*

5. Stop the application by pressing `Ctrl+C`.