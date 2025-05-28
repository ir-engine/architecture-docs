# Application configuration management

## Overview

Application Configuration Management is a system that provides a centralized approach to managing and accessing configuration settings for the iR Engine's background processing application. It enables the Task Server and its various tasks to retrieve operational parameters such as server ports, task intervals, database connection details, and other environment-specific settings. By separating configuration from code, this system enhances flexibility, maintainability, and operational clarity across different deployment environments.

## Purpose and functionality

Configuration management serves several essential purposes:

1. **Centralization**: Provides a single source of truth for application settings
2. **Environment adaptation**: Allows the application to behave differently in development, testing, and production
3. **Operational flexibility**: Enables changing application behavior without modifying code
4. **Consistency**: Ensures all components use the same configuration values
5. **Security**: Separates sensitive information from application code

The system manages various types of configuration values:

- **Server settings**: Ports, hostnames, and network parameters
- **Task parameters**: Execution intervals and processing limits
- **Integration details**: Connection information for databases, Redis, and other services
- **Environment-specific values**: Different settings for development, testing, and production
- **Operational parameters**: Logging levels, feature flags, and other runtime controls

## Implementation

### Configuration loading

The configuration loading process begins in the application's entry point:

```typescript
// src/index.ts
import { updateAppConfig } from '@ir-engine/server-core/src/updateAppConfig';

const init = async () => {
  // Load all configurations before proceeding
  await updateAppConfig();
  
  // Continue with application startup
  const { start } = await import('./start');
  start();
};

init();
```

The `updateAppConfig()` function:
1. Gathers configuration values from various sources
2. Merges them according to priority rules
3. Populates a central configuration object
4. Makes this object available through the `appconfig` module

### Configuration sources

The configuration system typically loads values from multiple sources:

1. **Default values**: Hardcoded defaults defined in the application
2. **Environment variables**: Values set in the server's environment
3. **Configuration files**: JSON or YAML files with settings
4. **Kubernetes ConfigMaps**: For Kubernetes deployments
5. **Secrets management**: For sensitive information like API keys

These sources are processed in order of priority, with later sources overriding earlier ones.

### Configuration access

Once loaded, configuration values are accessible throughout the application:

```typescript
// Example from a task file
import config from '@ir-engine/server-core/src/appconfig';
import multiLogger from '@ir-engine/server-core/src/ServerLogger';

const logger = multiLogger.child({ component: 'taskserver:collect-analytics' });

// Access configuration values
const intervalSeconds = config['task-server'].processInterval || 1800; // Default: 30 minutes
const intervalMilliseconds = intervalSeconds * 1000;

logger.info(`Analytics collection interval set to ${intervalSeconds} seconds`);

// Use the configuration value
setInterval(async () => {
  // Task logic here
}, intervalMilliseconds);
```

This pattern:
1. Imports the central configuration object
2. Accesses specific settings using property paths
3. Provides fallback values for missing configurations
4. Uses the retrieved values to control application behavior

### Configuration structure

The configuration object typically follows a structured format:

```typescript
// Simplified view of the configuration object
const config = {
  server: {
    port: 3030,
    hostname: 'localhost',
    namespace: 'ir-engine-dev'
  },
  'task-server': {
    port: 5050,
    processInterval: 1800
  },
  redis: {
    host: 'localhost',
    port: 6379
  },
  kubernetes: {
    inCluster: false,
    namespace: 'ir-engine-dev'
  },
  analytics: {
    enabled: true,
    storageLimit: 10000
  }
  // Additional sections for other components
};
```

This hierarchical structure:
- Groups related settings together
- Provides clear namespacing for different components
- Makes configuration values easy to locate and reference
- Allows for partial updates of specific sections

## Configuration workflow

The complete configuration management workflow follows these steps:

```mermaid
sequenceDiagram
    participant AppStart as Application Startup
    participant UpdateConfig as updateAppConfig()
    participant EnvVars as Environment Variables
    participant Defaults as Default Values
    participant ConfigObj as Central Config Object
    participant Tasks as Task Modules

    AppStart->>UpdateConfig: Call updateAppConfig()
    UpdateConfig->>Defaults: Load default values
    Defaults-->>UpdateConfig: Default configuration
    UpdateConfig->>EnvVars: Check for environment variables
    EnvVars-->>UpdateConfig: Environment-specific values
    UpdateConfig->>ConfigObj: Merge and populate config object
    ConfigObj-->>UpdateConfig: Configuration complete
    UpdateConfig-->>AppStart: Configuration loaded
    AppStart->>Tasks: Initialize tasks
    Tasks->>ConfigObj: import config; access settings
    ConfigObj-->>Tasks: Provide configuration values
    Tasks->>Tasks: Use values to control behavior
```

This diagram illustrates:
1. The application starts and calls `updateAppConfig()`
2. Default values are loaded as a baseline
3. Environment variables and other sources are checked
4. Values are merged with defaults taking lowest priority
5. The central configuration object is populated
6. Tasks and other components import and use the configuration
7. Configuration values control application behavior

## Practical examples

### Task interval configuration

The Analytics Data Collector uses configuration to determine how often it runs:

```typescript
// Simplified from src/collect-analytics.ts
import config from '@ir-engine/server-core/src/appconfig';

export default (app) => {
  // Get the configured interval or use default
  const intervalSeconds = config['task-server'].processInterval || 1800;
  const intervalMilliseconds = intervalSeconds * 1000;
  
  // Set up periodic execution
  const interval = setInterval(async () => {
    try {
      // Analytics collection logic
      await collectAndProcessAnalytics(app);
    } catch (err) {
      logger.error('Error collecting analytics:', err);
    }
  }, intervalMilliseconds);
  
  // Clean up on application shutdown
  app.on('close', () => {
    clearInterval(interval);
  });
};
```

This example shows how:
1. The task imports the configuration object
2. It retrieves the configured interval with a fallback
3. The configuration value directly controls the task's behavior
4. Changes to the configuration will affect the task's execution frequency

### Kubernetes namespace configuration

The Kubernetes Event Collector uses configuration to determine which namespace to monitor:

```typescript
// Simplified from src/collect-events.ts
import config from '@ir-engine/server-core/src/appconfig';

const collectKubernetesEvents = async (app) => {
  // Get the configured namespace
  const namespace = config.kubernetes.namespace || 'default';
  
  logger.info(`Collecting events from namespace: ${namespace}`);
  
  // Use the Kubernetes client to watch events
  const k8sApi = app.get('k8sApi');
  const watch = new k8s.Watch(k8sApi.config);
  
  watch.watch(
    `/api/v1/namespaces/${namespace}/events`,
    {},
    async (type, event) => {
      // Process Kubernetes events
    }
  );
};
```

This example demonstrates:
1. Retrieving a namespace configuration value
2. Using the value to determine which Kubernetes resources to monitor
3. Adapting the application's behavior based on configuration

## Deployment considerations

When deploying the application, configuration values can be provided in several ways:

### Environment variables

For containerized deployments, environment variables are commonly used:

```yaml
# Kubernetes deployment example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ir-engine-taskserver
spec:
  template:
    spec:
      containers:
      - name: taskserver
        image: ir-engine/taskserver:latest
        env:
        - name: NODE_ENV
          value: production
        - name: TASK_SERVER_PROCESS_INTERVAL
          value: "600"  # 10 minutes instead of default 30
        - name: KUBERNETES_NAMESPACE
          value: "ir-engine-prod"
```

### ConfigMaps in Kubernetes

For more complex configurations, Kubernetes ConfigMaps can be used:

```yaml
# ConfigMap definition
apiVersion: v1
kind: ConfigMap
metadata:
  name: ir-engine-config
data:
  task-server.json: |
    {
      "port": 5050,
      "processInterval": 600
    }
  analytics.json: |
    {
      "enabled": true,
      "storageLimit": 20000
    }

# Mounting in deployment
spec:
  containers:
  - name: taskserver
    volumeMounts:
    - name: config-volume
      mountPath: /app/config
  volumes:
  - name: config-volume
    configMap:
      name: ir-engine-config
```

## Benefits of centralized configuration

The Application Configuration Management system provides several key benefits:

1. **Separation of concerns**: Keeps operational parameters separate from application logic
2. **Environment adaptability**: Allows the same code to run in different environments
3. **Operational flexibility**: Enables changing behavior without code modifications
4. **Reduced errors**: Eliminates hardcoded values scattered throughout the codebase
5. **Simplified deployment**: Makes it easier to deploy the application in new environments
6. **Enhanced security**: Keeps sensitive information out of the codebase
7. **Improved maintainability**: Centralizes all configuration in one logical location

These benefits make configuration management a critical component of the iR Engine's background processing system.

## Next steps

With an understanding of how the Task Server Application starts and how it accesses configuration, the next chapter explores one of the primary background tasks: the Analytics Data Collector.

Next: [Analytics data collector](03_analytics_data_collector_.md)

---


