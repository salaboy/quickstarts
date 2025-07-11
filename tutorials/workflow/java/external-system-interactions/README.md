# External Events

This tutorial demonstrates how to author a workflow where the workflow will wait until it receives an external event. This pattern is often applied in workflows that require an approval step. For more information about the external system interaction pattern see the [Dapr docs](https://docs.dapr.io/developing-applications/building-blocks/workflow/workflow-patterns/#external-system-interaction).

## Inspect the code

Open the [`ExternalEventsWorkflow.java`](src/main/java/io/dapr/springboot/examples/external/ExternalEventsWorkflow.java) file in the `tutorials/workflow/java/external-system-interactions/src/main/java/io/dapr/springboot/examples/external/` folder. 
 This file contains the definition for the workflow. It is an order workflow that requests an external approval if the order has a total price greater than 250 dollars.

```mermaid
graph LR
   SW((Start
   Workflow))
   IF1{Is TotalPrice
    > 250?}
   IF2{Is Order Approved
   or TotalPrice < 250?}
   WAIT[Wait for
   approval event]
   EX{Event
   received?}
   PO[Process Order]
   SN[Send Notification]
   EW((End
   Workflow))
   SW --> IF1
   IF1 -->|Yes| WAIT
   IF1 -->|No| IF2
   EX -->|Yes| IF2
   EX -->|No| SN
   WAIT --> EX
   IF2 -->|Yes| PO
   PO --> SN
   IF2 -->|No| SN
   SN --> EW
```

## Run the tutorial

1. Use a terminal to navigate to the `tutorials/workflow/java/external-system-interactions` folder.
2. Build and run the project using Maven.

   ```bash
   mvn spring-boot:test-run
   ```

3. Use the POST request in the [`externalevents.http`](./externalevents.http) file to start the workflow, or use this cURL command:

    ```bash
    curl -i --request POST \
    --url http://localhost:8080/start \
    --header 'content-type: application/json' \
    --data '{"id": "b7dd836b-e913-4446-9912-d400befebec5","description": "Rubber ducks","quantity": 100,"totalPrice": 500}'
    ```

    The input for the workflow is an `Order` object:

    ```json
    {
        "id": "{{orderId}}",
        "description": "Rubber ducks",
        "quantity": 100,
        "totalPrice": 500
    }
    ```

4. Use the GET request in the [`externalevents.http`](./externalevents.http) file to get the status of the workflow, or use this cURL command:

    ```bash
    curl --request GET --url http://localhost:8080/status
    ```

    The workflow should still be running since it is waiting for an external event.

    The app logs should look similar to the following:

    ```text
   Received order: Order[id=b7dd836b-e913-4446-9912-d400befebec5, description=Rubber ducks, quantity=100, totalPrice=500.0]

    ```

5. Use the POST request in the [`externalevents.http`](./externalevents.http) file to send an `approval-event` to the workflow, or use this cURL command:

    ```bash
    curl -i --request POST \
    --url http://localhost:8080/event \
    --header 'content-type: application/json' \
    --data '{"orderId": "b7dd836b-e913-4446-9912-d400befebec5","isApproved": true}'
    ```

    The payload for the event is an `ApprovalStatus` object:

    ```json
    {
        "orderId": "{{instanceId}}",
        "isApproved": true
    }
    ```

    *The workflow will only wait for the external approval event for 2 minutes before timing out. In this case you will need to start a new order workflow instance.*

6. Again use the GET request in the [`externalevents.http`](./externalevents.http) file to get the status of the workflow. The workflow should now be completed.

    The expected serialized output of the workflow is:

    ```txt
    "\"Workflow Instance (32d6bae5-5823-4d2a-8113-02f5265d629d) Status: COMPLETED
       Output: Order b7dd836b-e913-4446-9912-d400befebec5 has been approved.\""
    ```

7. Stop the application by pressing `Ctrl+C`.
