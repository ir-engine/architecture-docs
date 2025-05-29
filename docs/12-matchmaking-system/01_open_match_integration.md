# Open match integration

Welcome to our exploration of iR Engine's matchmaking system! In this chapter, we'll dive into how iR Engine integrates with Google's Open Match framework to provide a powerful and scalable matchmaking solution.

## What is Open Match?

**Open Match** is an open-source matchmaking framework developed by Google and Unity. It's designed to be flexible, scalable, and customizable, making it perfect for connecting players in multiplayer games and virtual experiences.

At its core, Open Match provides:

1. A **ticket-based** system for tracking player matchmaking requests
2. A **component-based** architecture that separates concerns
3. **Kubernetes-native** deployment for scalability
4. **Customizable matching logic** through user-defined functions

## iR Engine's Open Match Integration

iR Engine integrates Open Match through the `@ir-engine/matchmaking` package. This package provides:

1. Client-side APIs for creating and tracking match tickets
2. Server-side services for managing the matchmaking process
3. Custom matchmaking functions for iR Engine's specific needs
4. A director component for assigning matches to instanceservers

Let's look at the structure of the matchmaking package:

```
packages/matchmaking/
├── src/
│   ├── functions/           # Client-side matchmaking functions
│   ├── match-ticket.schema.ts # Match ticket schema definition
│   └── ...
├── open-match-custom-pods/  # Custom Open Match components
│   ├── director/            # Match director implementation
│   │   └── main.go          # Go implementation of the director
│   └── matchfunction/       # Matchmaking function implementation
│       └── mmf/             # Match function logic
│           ├── matchfunction.go
│           └── server.go
└── package.json
```

## How Open Match Works in iR Engine

When a player wants to join a multiplayer experience in iR Engine, the following sequence occurs:

1. The client creates a match ticket with player preferences
2. The ticket is stored in Open Match's ticket pool
3. The matchmaking function periodically processes tickets to form matches
4. The director assigns matches to instanceservers
5. Players are notified of their match assignment
6. Players connect to the assigned instanceserver

Let's look at how a match ticket is created:

```typescript
// Simplified from packages/matchmaking/src/functions.ts
export const createTicket = async (gameMode: string): Promise<MatchTicketType> => {
  // Create a ticket in the database
  const ticket = await API.instance.service(matchTicketPath).create({
    gameMode
  })

  return ticket
}
```

## Open Match Components in iR Engine

iR Engine implements several custom Open Match components:

### 1. Matchmaking Function (MMF)

The matchmaking function is implemented in Go and deployed as a Kubernetes pod. It's responsible for:

- Querying the ticket pool for available tickets
- Grouping tickets into potential matches based on game mode and other criteria
- Calculating a score for each potential match
- Submitting matches to the Open Match backend

```go
// Simplified from packages/matchmaking/open-match-custom-pods/matchfunction/mmf/matchfunction.go
func makeMatches(p *pb.MatchProfile, poolTickets map[string][]*pb.Ticket) ([]*pb.Match, error) {
    // Group tickets by game mode
    ticketGroups := groupTicketsByGameMode(poolTickets)

    // Create matches from ticket groups
    var matches []*pb.Match
    for gameMode, tickets := range ticketGroups {
        // Create matches with appropriate team sizes
        // ...

        // Calculate match score
        matchScore := scoreCalculator(matchTickets)

        // Create the match
        matches = append(matches, &pb.Match{
            MatchId:       fmt.Sprintf("profile-%v-time-%v-%v", p.GetName(), time.Now().Format("2006-01-02T15:04:05.00"), count),
            MatchProfile:  p.GetName(),
            MatchFunction: matchName,
            Tickets:       matchTickets,
            // ...
        })
    }

    return matches, nil
}
```

### 2. Director

The director is also implemented in Go and deployed as a Kubernetes pod. It's responsible for:

- Fetching matches from the Open Match backend
- Assigning matches to instanceservers
- Notifying players of their match assignment

```go
// Simplified from packages/matchmaking/open-match-custom-pods/director/main.go
func assign(be pb.BackendServiceClient, p *pb.MatchProfile, matches []*pb.Match) error {
    for _, match := range matches {
        // Get ticket IDs from the match
        var ticketIDs []string
        for _, ticket := range match.GetTickets() {
            ticketIDs = append(ticketIDs, ticket.GetId())
        }

        // Generate a connection string (instanceserver ID)
        conn := uuid.New().String()

        // Assign tickets to the connection
        req := &pb.AssignTicketsRequest{
            Assignments: []*pb.AssignmentGroup{
                {
                    TicketIds: ticketIDs,
                    Assignment: &pb.Assignment{
                        Connection: conn,
                        // ...
                    },
                },
            },
        }

        // Make the assignment
        if _, err := be.AssignTickets(context.Background(), req); err != nil {
            return fmt.Errorf("AssignTickets failed for match %v, got %w", match.GetMatchId(), err)
        }
    }

    return nil
}
```

## Deploying Open Match with iR Engine

iR Engine provides scripts and Helm charts for deploying Open Match on Kubernetes:

```bash
# From packages/matchmaking/README.md
# Start minikube with ingress
minikube start --addons ingress

# Install Open Match with Helm
helm install --set frontend.host=<hostname> open-match packages/ops/open-match

# Build and deploy the custom pods
helm install -f path/to/matchmaking.yaml --set director.image.tag=<TAG>,matchfunction.image.tag=<TAG> <release>-matchmaking ../ops/etherealengine-matchmaking
```

## Integration with iR Engine's Server Core

The matchmaking system integrates with iR Engine's server-core through several services:

1. `MatchTicketService`: Manages match tickets in the database
2. `MatchTicketAssignmentService`: Handles match assignments
3. `MatchUserService`: Associates users with match tickets
4. `MatchInstanceService`: Manages match instances

These services provide the bridge between iR Engine's backend and the Open Match framework.

## Conclusion

iR Engine's integration with Open Match provides a powerful, scalable matchmaking solution that can handle a wide variety of matchmaking scenarios. By leveraging Open Match's flexible architecture and Kubernetes-native deployment, iR Engine can match players efficiently while maintaining full control over the matchmaking logic.

In the next chapter, we'll explore the lifecycle of a match ticket, from creation to assignment.

---


