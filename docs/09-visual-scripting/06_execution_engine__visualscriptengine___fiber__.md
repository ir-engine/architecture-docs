# Execution engine (VisualScriptEngine & Fiber)

## Overview

The execution engine is the runtime system that brings visual scripts to life. It transforms the static structure of nodes, sockets, and links into dynamic, running programs by managing execution flow, resolving data dependencies, and coordinating asynchronous operations. 

The iR Engine's execution engine consists of two primary components: the VisualScriptEngine, which orchestrates the overall execution process, and Fibers, which represent individual execution paths through the graph. This chapter explores the concept, structure, and implementation of the execution engine within the iR Engine.

## Core concepts

### Runtime execution

While the visual script graph defines the structure of a program, the execution engine determines how that program runs:

1. **Initialization**: The engine prepares the graph for execution by setting up event nodes
2. **Event handling**: When events occur, the engine initiates execution flows from corresponding event nodes
3. **Sequential processing**: The engine follows the connections between nodes to determine execution order
4. **Data resolution**: Before executing a node, the engine ensures all required input data is available
5. **Asynchronous management**: The engine handles operations that take time without blocking the entire script

This runtime system enables visual scripts to respond to events, process data, and perform actions in a controlled, predictable manner.

### Execution components

The execution engine consists of two primary components that work together:

1. **VisualScriptEngine**: The central coordinator that:
   - Manages the overall execution process
   - Maintains references to all nodes in the graph
   - Initializes event nodes
   - Handles asynchronous operations
   - Coordinates multiple execution paths

2. **Fiber**: An individual execution path that:
   - Represents a single thread of execution through the graph
   - Moves step-by-step from one node to the next
   - Resolves input data for nodes
   - Triggers node execution
   - Follows output connections to determine the next node

These components work together to ensure that visual scripts execute correctly, with nodes running in the proper sequence and with the necessary data.

## Implementation

### VisualScriptEngine

The `VisualScriptEngine` class serves as the central coordinator for script execution:

```typescript
// Simplified from: src/engine/Execution/VisualScriptEngine.ts
export class VisualScriptEngine {
  public readonly eventNodes: IEventNode[] = [];
  public readonly asyncNodes: IAsyncNode[] = [];
  private fiberQueue: Fiber[] = [];
  
  constructor(public readonly nodes: Record<string, INode>) {
    // Find all event nodes in the graph
    Object.values(nodes).forEach((node) => {
      if (isEventNode(node)) {
        this.eventNodes.push(node);
      }
    });
    
    // Initialize all event nodes
    this.eventNodes.forEach((eventNode) => {
      eventNode.init(this);
    });
  }
  
  // Start a new execution path from an event or async node
  public commitToNewFiber(
    node: INode,
    outputFlowSocketName: string,
    onComplete?: () => void
  ): void {
    const outputSocket = node.outputs.find(s => s.name === outputFlowSocketName);
    if (outputSocket && outputSocket.links.length > 0) {
      const link = outputSocket.links[0];
      const fiber = new Fiber(this, link, onComplete);
      this.fiberQueue.push(fiber);
    }
  }
  
  // Execute all pending fibers synchronously
  public executeAllSync(maxSteps: number = Infinity): number {
    let stepsExecuted = 0;
    
    while (stepsExecuted < maxSteps && this.fiberQueue.length > 0) {
      const currentFiber = this.fiberQueue[0];
      stepsExecuted += currentFiber.executeStep();
      
      if (currentFiber.isCompleted()) {
        this.fiberQueue.shift();
      }
    }
    
    return stepsExecuted;
  }
  
  // Register an async operation
  public registerAsyncOperation(
    node: IAsyncNode,
    onComplete: () => void
  ): void {
    this.asyncNodes.push(node);
    // Set up completion callback
    const originalOnComplete = onComplete;
    onComplete = () => {
      // Remove from async nodes list
      const index = this.asyncNodes.indexOf(node);
      if (index >= 0) {
        this.asyncNodes.splice(index, 1);
      }
      // Call original callback
      originalOnComplete();
    };
    
    // Start the async operation
    node.startAsync(onComplete);
  }
}
```

The `VisualScriptEngine`:
- Collects and initializes all event nodes during construction
- Provides methods to start new execution paths (`commitToNewFiber`)
- Manages the queue of active fibers
- Executes fibers in sequence
- Handles registration and completion of asynchronous operations

### Fiber

The `Fiber` class represents a single execution path through the graph:

```typescript
// Simplified from: src/engine/Execution/Fiber.ts
export class Fiber {
  private stepsExecuted: number = 0;
  
  constructor(
    public readonly engine: VisualScriptEngine,
    public nextEval: Link | null,
    private onComplete?: () => void
  ) {}
  
  // Execute one step in this fiber
  public executeStep(): number {
    if (!this.nextEval) {
      this.complete();
      return 0;
    }
    
    const link = this.nextEval;
    this.nextEval = null;
    
    const targetNode = this.engine.nodes[link.nodeId];
    const inputSocketName = link.socketName;
    
    // Resolve all data inputs for the target node
    let stepsForInputs = 0;
    targetNode.inputs.forEach(inputSocket => {
      if (inputSocket.valueTypeName !== 'flow') {
        stepsForInputs += resolveSocketValue(this.engine, inputSocket);
      }
    });
    
    // Execute the node
    if (isFlowNode(targetNode)) {
      targetNode.triggered(this, inputSocketName);
    } else if (isAsyncNode(targetNode)) {
      this.engine.registerAsyncOperation(targetNode, () => {
        // When async operation completes, continue from its output
        this.engine.commitToNewFiber(
          targetNode,
          targetNode.getOutputFlowSocketName(),
          this.onComplete
        );
      });
    }
    
    this.stepsExecuted++;
    return stepsForInputs + 1;
  }
  
  // Called by a flow node to specify the next node to execute
  public commit(node: INode, outputSocketName: string): void {
    const outputSocket = node.outputs.find(s => s.name === outputSocketName);
    if (outputSocket && outputSocket.links.length > 0) {
      this.nextEval = outputSocket.links[0];
    } else {
      this.complete();
    }
  }
  
  // Check if this fiber has completed
  public isCompleted(): boolean {
    return this.nextEval === null;
  }
  
  // Mark this fiber as complete and call the completion callback
  private complete(): void {
    if (this.onComplete) {
      this.onComplete();
    }
  }
}
```

The `Fiber`:
- Tracks the next link to evaluate
- Resolves input data for nodes before execution
- Executes nodes and follows their output connections
- Provides a `commit` method for nodes to specify the next execution step
- Handles completion of the execution path

### Data resolution

Before executing a node, the system must ensure all its input data is available. This is handled by the `resolveSocketValue` function:

```typescript
// Simplified from: src/engine/Execution/resolveSocketValue.ts
export function resolveSocketValue(
  engine: VisualScriptEngine,
  inputSocket: Socket
): number {
  // If the socket has no incoming links, use its current value
  if (inputSocket.links.length === 0) {
    return 0;
  }
  
  // Get the source of the input value
  const sourceLink = inputSocket.links[0];
  const sourceNode = engine.nodes[sourceLink.nodeId];
  const sourceSocket = sourceNode.outputs.find(s => s.name === sourceLink.socketName);
  
  // If the source is a function node, execute it to get the value
  if (isFunctionNode(sourceNode)) {
    // Recursively resolve inputs for the function node
    let stepsForFunctionInputs = 0;
    sourceNode.inputs.forEach(fnInputSocket => {
      stepsForFunctionInputs += resolveSocketValue(engine, fnInputSocket);
    });
    
    // Execute the function node
    sourceNode.exec();
    
    // Copy the output value to the input socket
    inputSocket.value = sourceSocket.value;
    
    return stepsForFunctionInputs + 1;
  } else {
    // For other node types, just copy the current output value
    inputSocket.value = sourceSocket.value;
    return 0;
  }
}
```

This function:
- Checks if the input socket has an incoming connection
- If connected to a function node, recursively resolves its inputs and executes it
- Copies the resulting value to the input socket
- Returns the number of execution steps performed

## Execution workflow

Let's examine the execution process for a simple "Hello, World!" script:

```mermaid
graph TD
    A[Event: On Game Start] --> B[Action: Print "Hello"]
    B --> C[Async Action: Wait 1 Sec]
    C --> D[Action: Print "World!"]
```

The execution follows these steps:

1. **Initialization**:
   - The `VisualScriptEngine` is created with the graph
   - It finds the `On Game Start` event node and initializes it
   - The event node sets up listeners for the game start event

2. **Event triggering**:
   - The game starts, triggering the `On Game Start` node
   - The node calls `engine.commitToNewFiber()` with its output socket

3. **First fiber creation**:
   - The engine creates a new `Fiber` starting at the link to `Print "Hello"`
   - The fiber is added to the engine's queue

4. **Executing "Print Hello"**:
   - The engine calls `fiber.executeStep()`
   - The fiber resolves any inputs for the `Print "Hello"` node
   - The fiber calls `node.triggered()` on the print node
   - The print node displays "Hello" and calls `fiber.commit()` with its output socket
   - The fiber updates its `nextEval` to the link to `Wait 1 Sec`

5. **Executing "Wait 1 Sec"**:
   - The engine calls `fiber.executeStep()` again
   - The fiber resolves inputs for the `Wait 1 Sec` node
   - The fiber identifies it as an async node and registers it with the engine
   - The wait node starts its timer
   - The current fiber is effectively completed (no `nextEval`)

6. **Asynchronous waiting**:
   - The engine continues processing other fibers or tasks
   - After 1 second, the wait node's timer completes
   - The wait node's completion callback is triggered

7. **New fiber creation**:
   - The completion callback calls `engine.commitToNewFiber()` with the wait node's output
   - The engine creates a new fiber starting at the link to `Print "World!"`

8. **Executing "Print World!"**:
   - The engine processes the new fiber
   - The fiber resolves inputs for the `Print "World!"` node
   - The fiber calls `node.triggered()` on the print node
   - The print node displays "World!" and calls `fiber.commit()`
   - With no further connections, the fiber completes

This sequence demonstrates how the execution engine manages both synchronous flow (from one node directly to the next) and asynchronous operations (waiting for a timer).

## Node execution

Different types of nodes interact with the execution engine in specific ways:

### Event node execution

```typescript
// Simplified concept for event nodes
class EventNode extends Node<NodeType.Event> {
  init(engine: VisualScriptEngine): void {
    // Set up event listener
    const eventSource = getEventSource();
    eventSource.addEventListener('event', () => {
      // When event occurs, start execution from this node
      engine.commitToNewFiber(this, 'output');
    });
  }
}
```

Event nodes:
- Initialize by setting up event listeners
- Start new execution paths when events occur
- Have no input execution sockets, only output execution sockets

### Flow node execution

```typescript
// Simplified concept for flow nodes
class FlowNode extends Node<NodeType.Flow> {
  triggered(fiber: Fiber, inputSocketName: string): void {
    // Read input values
    const inputValue = this.readInput<string>('message');
    
    // Perform node-specific action
    console.log(inputValue);
    
    // Continue execution to the next node
    fiber.commit(this, 'next');
  }
}
```

Flow nodes:
- Are triggered by a fiber through their input execution socket
- Read input values, perform their action, and determine the next step
- Call `fiber.commit()` to specify which output execution path to follow

### Async node execution

```typescript
// Simplified concept for async nodes
class AsyncNode extends Node<NodeType.Async> {
  triggered(engine: VisualScriptEngine, inputSocketName: string, onComplete: () => void): void {
    // Read input values
    const duration = this.readInput<number>('duration');
    
    // Register the async operation
    engine.registerAsyncOperation(this, onComplete);
  }
  
  startAsync(onComplete: () => void): void {
    // Start the asynchronous operation
    setTimeout(onComplete, this.readInput<number>('duration') * 1000);
  }
  
  getOutputFlowSocketName(): string {
    // Specify which output socket to follow when complete
    return 'completed';
  }
}
```

Async nodes:
- Register themselves with the engine for asynchronous handling
- Start their operation (e.g., timer, file loading) without blocking execution
- Specify which output path to follow when the operation completes

### Function node execution

```typescript
// Simplified concept for function nodes
class FunctionNode extends Node<NodeType.Function> {
  exec(): void {
    // Read input values
    const a = this.readInput<number>('a');
    const b = this.readInput<number>('b');
    
    // Perform calculation
    const result = a + b;
    
    // Write to output
    this.writeOutput<number>('sum', result);
  }
}
```

Function nodes:
- Are executed on-demand when their output is needed
- Read input values, perform calculations, and write results to outputs
- Do not participate directly in execution flow

## Next steps

With an understanding of how the execution engine processes visual scripts at runtime, the next chapter explores how the system manages the available node types and value types through the node and value registry.

Next: [Node & value registry](07_node___value_registry_.md)

---


