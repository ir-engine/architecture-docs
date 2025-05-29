# Instanceserver architecture

Welcome to the first chapter of our exploration into iR Engine's multiplayer infrastructure! In this chapter, we'll dive into the architecture of the instanceserver, which is the backbone of iR Engine's multiplayer capabilities.

## What is an Instanceserver?

In the world of multiplayer games and virtual worlds, an **instanceserver** is a dedicated server that hosts a specific instance of a virtual environment. Think of it as a single "room" or "world" where multiple users can connect and interact with each other in real-time.

In iR Engine, the instanceserver is a specialized application that:

1. Hosts a specific location or scene
2. Manages connections from multiple clients
3. Synchronizes state between all connected users
4. Handles physics simulations
5. Processes voice and video communication
6. Authenticates users and manages sessions

## Instanceserver Package Structure

The instanceserver is implemented in the `@ir-engine/instanceserver` package. Let's look at its main components:

```
packages/instanceserver/
├── src/
│   ├── index.ts                 # Entry point
│   ├── start.ts                 # Server startup logic
│   ├── InstanceServerState.ts   # State management
│   ├── channels.ts              # Communication channels
│   ├── NetworkFunctions.ts      # Network handling
│   ├── SocketFunctions.ts       # Socket management
│   ├── SocketWebRTCServerFunctions.ts # WebRTC implementation
│   └── restartInstanceServer.ts # Server restart logic
└── package.json
```

## How the Instanceserver Works

When a user wants to join a virtual world in iR Engine, the following sequence occurs:

1. The client requests to join a specific location
2. The server-core provisions an instanceserver (or assigns an existing one)
3. The instanceserver initializes the scene and spatial engine
4. The client connects to the instanceserver via WebRTC
5. The instanceserver authenticates the user
6. The client joins the virtual world and begins receiving state updates

Let's look at the initialization process in more detail:

```typescript
// Simplified from packages/instanceserver/src/start.ts
export const start = async (): Promise<Application> => {
  // Create the Feathers application
  const app = await createFeathersKoaApp(ServerMode.Instance, instanceServerPipe)

  // Initialize the spatial timer for physics and animations
  startTimer()

  // Connect to Agones for Kubernetes orchestration
  const agonesSDK = new AgonesSDK()
  agonesSDK.connect()
  agonesSDK.ready()

  // Set up health checks
  setInterval(() => agonesSDK.health(), 1000)

  // Configure communication channels
  app.configure(channels)

  // Initialize the network
  initializeNetwork()

  return app
}
```

## InstanceServerState

The instanceserver maintains its state using Hyperflux, iR Engine's state management system. The `InstanceServerState` contains information about:

- The current instance ID
- The location being hosted
- Connected users
- Network status
- Server configuration

```typescript
// Simplified from packages/instanceserver/src/InstanceServerState.ts
export const InstanceServerState = defineState({
  name: 'InstanceServerState',
  initial: {
    instance: null as InstanceType,
    currentScene: null as SceneData,
    channelId: null as ChannelID,
    channelType: null as ChannelType,
    videoEnabled: false,
    instanceProvisioned: false,
    instanceProvisionId: null as InstanceProvisionID
  }
})
```

## Integration with the Engine

The instanceserver integrates deeply with iR Engine's core systems:

1. **ECS (Entity Component System)**: The instanceserver uses the same ECS architecture as the client, allowing for seamless state synchronization.

2. **Spatial Engine**: The instanceserver runs the spatial engine to handle physics, transformations, and other spatial operations.

3. **Network**: The instanceserver implements network protocols for efficient state synchronization.

4. **Hyperflux**: State management is handled through Hyperflux, allowing for reactive updates.

## Scaling with Kubernetes and Agones

One of the most powerful aspects of the instanceserver architecture is its ability to scale dynamically using Kubernetes and Agones:

1. **Kubernetes** provides the container orchestration platform
2. **Agones** extends Kubernetes with game server-specific features
3. The instanceserver registers with Agones on startup
4. Agones manages the lifecycle of instanceserver pods
5. New instances can be provisioned on demand

This architecture allows iR Engine to scale from a few users to thousands, with each instanceserver handling a specific location or scene.

## Conclusion

The instanceserver is the foundation of iR Engine's multiplayer capabilities. It provides a robust, scalable platform for hosting virtual worlds and enabling real-time interactions between users.

In the next chapter, we'll explore how WebRTC networking enables low-latency communication between clients and the instanceserver.

---


