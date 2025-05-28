# Background processing

The iR Engine's background processing system is designed to run automated tasks at regular intervals. Key functions include periodically collecting analytics data about platform usage (such as active users and instances) and gathering event logs from the Kubernetes cluster where the application is deployed. This helps in monitoring the system and gathering operational insights.

## Architecture overview

The background processing system consists of several interconnected components that work together to handle various automated tasks:

```mermaid
flowchart TD
    A0["Task server application"]
    A1["Periodic task scheduler"]
    A2["Analytics data collector"]
    A3["Kubernetes event collector"]
    A4["Application configuration management"]
    A5["Service interaction layer"]
    A0 -- "Initializes and uses" --> A4
    A0 -- "Launches task" --> A2
    A0 -- "Launches task" --> A3
    A2 -- "Uses for scheduling" --> A1
    A2 -- "Interacts via services" --> A5
    A3 -- "Reads configuration" --> A4
```

## Documentation chapters

1. [Task server application](01_task_server_application_.md)
2. [Application configuration management](02_application_configuration_management_.md)
3. [Analytics data collector](03_analytics_data_collector_.md)
4. [Periodic task scheduler](04_periodic_task_scheduler_.md)
5. [Service interaction layer](05_service_interaction_layer_.md)
6. [Kubernetes event collector](06_kubernetes_event_collector_.md)

---


