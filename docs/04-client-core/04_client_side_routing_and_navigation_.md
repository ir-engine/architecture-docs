# Client-side routing and navigation

## Overview

The Client-Side Routing and Navigation system enables users to move between different views or "pages" within the iR Engine client without requiring full page reloads. It creates a seamless, application-like experience by dynamically swapping content based on the current URL path. By leveraging React Router and a custom RouterService, the system provides both declarative route definitions and programmatic navigation capabilities. This chapter explores the implementation, workflow, and integration of client-side routing within the iR Engine client.

## Core concepts

### Single page applications

Traditional websites typically load a new HTML page from the server for each URL the user visits. In contrast, Single Page Applications (SPAs) like the iR Engine client:

1. **Load once**: The application loads a single HTML page initially
2. **Dynamic content**: JavaScript dynamically updates the content as users navigate
3. **No page reloads**: Navigation between "pages" happens without refreshing the browser
4. **State preservation**: Application state can be maintained across views

This approach provides several benefits:
- Faster navigation between views
- Smoother transitions and animations
- Persistent state across different sections
- Reduced server load for page rendering

### Client-side routing

Client-side routing is the mechanism that enables SPAs to:

1. **Match URLs to views**: Associate URL paths with specific components to render
2. **Handle navigation**: Update the browser's address bar without page reloads
3. **Manage history**: Support browser back/forward buttons and bookmarking
4. **Control access**: Restrict navigation based on authentication or permissions

The iR Engine client implements these capabilities using React Router and a custom RouterService.

## Implementation

### Route definition

Routes are defined using React Router's components:

```jsx
// Simplified example of route definitions
import { Routes, Route } from 'react-router-dom';
import HomePage from './pages/HomePage';
import ProfilePage from './pages/ProfilePage';
import AdminDashboard from './admin/AdminDashboard';

function AppRoutes() {
  return (
    <Routes>
      <Route path="/" element={<HomePage />} />
      <Route path="/profile" element={<ProfilePage />} />
      <Route path="/admin" element={<AdminDashboard />} />
      {/* Additional routes */}
    </Routes>
  );
}
```

This code:
- Imports the necessary components from React Router
- Defines a component that contains route definitions
- Maps URL paths to specific React components
- Creates a routing structure for the application

When a user navigates to a specific URL, React Router matches the path and renders the corresponding component.

### Navigation links

Users navigate between routes using the `Link` component:

```jsx
// Example of navigation links
import { Link } from 'react-router-dom';

function NavigationBar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/profile">My Profile</Link>
      <Link to="/admin">Admin Area</Link>
    </nav>
  );
}
```

The `Link` component:
- Renders as an HTML anchor (`<a>`) element
- Intercepts clicks to prevent default browser navigation
- Updates the URL using the History API
- Triggers React Router to render the matching component

This creates a navigation experience that feels like traditional links but avoids full page reloads.

### RouterService

The iR Engine client extends React Router with a custom RouterService that provides programmatic navigation:

```typescript
// Simplified from src/common/services/RouterService.tsx
import { createBrowserHistory, History } from 'history';
import { defineState } from '@ir-engine/hyperflux';

// Create a history object to manage browser session history
export const history: History = createBrowserHistory();

// Define a Hyperflux state for router actions
export const RouterState = defineState({
  name: 'RouterState',
  initial: {},
  
  // Action to navigate programmatically
  navigate: (pathname: string, searchParams = {}) => {
    // Convert searchParams object to URL query string if provided
    let search = '';
    if (Object.keys(searchParams).length > 0) {
      const params = new URLSearchParams();
      for (const [key, value] of Object.entries(searchParams)) {
        params.append(key, String(value));
      }
      search = `?${params.toString()}`;
    }
    
    // Update browser history and trigger route change
    history.push({
      pathname,
      search
    });
  }
});
```

This service:
- Creates a custom history object using the `history` library
- Defines a Hyperflux state with a `navigate` action
- Provides a convenient method to trigger navigation from anywhere in the application
- Supports adding query parameters to the URL

### Redirect component

For declarative redirects, the iR Engine client includes a Redirect component:

```typescript
// Simplified from src/common/components/Redirect.tsx
import React, { useEffect } from 'react';
import { RouterState } from '../services/RouterService';

export const Redirect = (props: { to: string }) => {
  useEffect(() => {
    // Navigate to the specified path when the component mounts
    RouterState.navigate(props.to);
  }, [props.to]);
  
  // This component doesn't render anything visible
  return null;
};
```

This component:
- Takes a destination path as a prop
- Uses the `useEffect` hook to trigger navigation when mounted
- Provides a declarative way to redirect users in JSX

### Integration with React Router

The RouterService integrates with React Router through a custom router setup:

```jsx
// Simplified concept of how the router is configured
import React from 'react';
import { Router } from 'react-router-dom';
import { history } from './common/services/RouterService';
import AppRoutes from './AppRoutes';

function MainApplication() {
  return (
    // Use Router with our custom history object
    <Router location={history.location} navigator={history}>
      <AppRoutes />
    </Router>
  );
}
```

This setup:
- Uses React Router's low-level `Router` component
- Connects it to our custom history object
- Ensures that both React Router and RouterService use the same history instance
- Enables consistent navigation behavior throughout the application

## Navigation workflow

The process of client-side navigation follows this sequence:

```mermaid
sequenceDiagram
    participant User
    participant Link as Link Component
    participant RouterService as RouterService
    participant History as Browser History API
    participant Router as React Router
    participant Component as Target Component

    User->>Link: Clicks navigation link
    Link->>RouterService: Intercepts click event
    RouterService->>History: Updates URL (history.push)
    History-->>RouterService: Confirms URL change
    RouterService->>Router: Notifies of location change
    Router->>Router: Matches new URL to route definition
    Router->>Component: Renders matching component
    Component-->>User: Displays new view
```

This diagram illustrates:
1. The user clicks a navigation link
2. The RouterService updates the browser URL without a page reload
3. React Router detects the URL change and finds the matching route
4. The corresponding component is rendered, updating the view

For programmatic navigation (using `RouterState.navigate()`), the process is similar but starts from code rather than a user click.

## Protected routes

The routing system integrates with authentication to create protected routes:

```jsx
// Example of a protected route component
import React, { useEffect } from 'react';
import { useMutableState } from '@ir-engine/hyperflux';
import { AuthState } from '../user/services/AuthState';
import { RouterState } from '../common/services/RouterService';

function ProtectedRoute({ children }) {
  const authState = useMutableState(AuthState);
  const isAuthenticated = authState.isAuthenticated.value;
  
  useEffect(() => {
    if (!isAuthenticated) {
      // Redirect to login if not authenticated
      RouterState.navigate('/login', { 
        redirectUrl: window.location.pathname 
      });
    }
  }, [isAuthenticated]);
  
  // Only render children if authenticated
  return isAuthenticated ? children : null;
}

// Usage in route definitions
function AppRoutes() {
  return (
    <Routes>
      <Route path="/" element={<HomePage />} />
      <Route path="/profile" element={
        <ProtectedRoute>
          <ProfilePage />
        </ProtectedRoute>
      } />
      {/* Other routes */}
    </Routes>
  );
}
```

This implementation:
- Checks authentication status using Hyperflux state
- Redirects unauthenticated users to the login page
- Includes the original destination as a query parameter for post-login redirection
- Only renders the protected content if the user is authenticated

## Nested routes

For complex sections like the admin panel, the iR Engine client uses nested routes:

```jsx
// Simplified example of nested routes in the admin panel
import React from 'react';
import { Routes, Route, Link } from 'react-router-dom';

// Admin panel components
const UserManagement = () => <div>User Management</div>;
const SiteSettings = () => <div>Site Settings</div>;
const AdminDashboard = () => <div>Admin Dashboard</div>;

function AdminRoutes() {
  return (
    <div className="admin-layout">
      <nav className="admin-sidebar">
        <Link to="/admin">Dashboard</Link>
        <Link to="/admin/users">Users</Link>
        <Link to="/admin/settings">Settings</Link>
      </nav>
      
      <div className="admin-content">
        <Routes>
          <Route path="/" element={<AdminDashboard />} />
          <Route path="users" element={<UserManagement />} />
          <Route path="settings" element={<SiteSettings />} />
        </Routes>
      </div>
    </div>
  );
}

// In the main routes file
function AppRoutes() {
  return (
    <Routes>
      <Route path="/" element={<HomePage />} />
      <Route path="/admin/*" element={<AdminRoutes />} />
      {/* Other routes */}
    </Routes>
  );
}
```

This approach:
- Uses the `/*` wildcard to match all paths under `/admin`
- Defines a separate set of routes within the admin component
- Maintains a consistent admin layout while changing only the content area
- Allows for modular organization of complex sections

## Integration with other components

The routing system integrates with several other components of the iR Engine client:

### Hyperflux state management

The RouterService uses Hyperflux for state management:

```typescript
// Example of routing state in Hyperflux
import { defineState, getMutableState } from '@ir-engine/hyperflux';

// Define routing state
export const RouterState = defineState({
  name: 'RouterState',
  initial: {
    currentPath: window.location.pathname,
    previousPath: null
  },
  
  // Navigation action
  navigate: (pathname) => {
    // Update state before navigation
    const state = getMutableState(RouterState);
    state.previousPath.set(state.currentPath.value);
    state.currentPath.set(pathname);
    
    // Perform actual navigation
    history.push(pathname);
  }
});
```

This integration:
- Tracks the current and previous paths in Hyperflux state
- Allows components to react to route changes
- Provides a centralized way to manage navigation state

### Authentication and authorization

Routing works closely with authentication to control access:

```typescript
// Example of route access control in the admin panel
import { useEffect } from 'react';
import { useFind } from '@ir-engine/common';
import { scopePath } from '@ir-engine/common/src/schema.type.module';
import { useMutableState } from '@ir-engine/hyperflux';
import { AllowedAdminRoutesState } from './AllowedAdminRoutesState';
import { RouterState } from '../common/services/RouterService';

function AdminAccessCheck() {
  const currentUserID = Engine.instance.userID;
  
  // Fetch user scopes from the server
  const scopeQuery = useFind(scopePath, {
    query: { userId: currentUserID }
  });
  
  useEffect(() => {
    if (scopeQuery.data) {
      // Check if the user has admin permission
      const isAdmin = scopeQuery.data.some(scope => 
        scope.type === 'admin:admin'
      );
      
      if (!isAdmin) {
        // Redirect unauthorized users
        RouterState.navigate('/');
      }
    }
  }, [scopeQuery.data]);
  
  // Render loading or admin content
  return scopeQuery.data ? (
    scopeQuery.data.some(scope => scope.type === 'admin:admin') ? (
      <AdminPanel />
    ) : (
      <div>Unauthorized. Redirecting...</div>
    )
  ) : (
    <div>Loading...</div>
  );
}
```

This integration:
- Fetches user permissions using FeathersJS services
- Checks if the user has the required permissions
- Redirects unauthorized users to an appropriate page
- Controls access to protected sections of the application

### URL parameters and query strings

The routing system supports URL parameters and query strings:

```jsx
// Example of route with parameters
<Route path="/user/:userId" element={<UserProfile />} />

// In the UserProfile component
import { useParams, useSearchParams } from 'react-router-dom';

function UserProfile() {
  // Get URL parameters
  const { userId } = useParams();
  
  // Get query parameters
  const [searchParams] = useSearchParams();
  const tab = searchParams.get('tab') || 'profile';
  
  return (
    <div>
      <h1>User Profile: {userId}</h1>
      <div className="tabs">
        <button className={tab === 'profile' ? 'active' : ''}>
          Profile
        </button>
        <button className={tab === 'settings' ? 'active' : ''}>
          Settings
        </button>
      </div>
      {/* Render content based on tab */}
    </div>
  );
}
```

This functionality:
- Extracts dynamic values from the URL path
- Retrieves optional parameters from the query string
- Enables dynamic content based on URL parameters
- Supports bookmarkable application states

## Benefits of client-side routing

The Client-Side Routing and Navigation system provides several key advantages:

1. **Improved performance**: Faster navigation without full page reloads
2. **Enhanced user experience**: Smooth transitions between views
3. **State preservation**: Maintains application state across navigation
4. **Deep linking**: Supports bookmarkable URLs for specific application states
5. **Code organization**: Structures the application into logical sections
6. **Access control**: Integrates with authentication to protect sensitive areas
7. **Familiar navigation**: Preserves expected browser behavior like back/forward buttons

These benefits make client-side routing an essential component of the iR Engine client architecture.

## Next steps

With an understanding of how users navigate between different views, the next chapter explores how the application creates user interfaces within the 3D environment.

Next: [XRUI and in-world widgets](05_xrui_and_in_world_widgets_.md)

---


