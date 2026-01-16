# AGENTS.md - Development Guidelines for pwm-2025-2

This document provides essential guidelines for agentic coding agents operating in this multi-project educational repository. It covers build commands, code style guidelines, and project-specific conventions for both Next.js and Expo React Native projects.

## Repository Overview

This is an educational repository for "Programação Web e Mobile" (Web and Mobile Programming) at UNICAP containing two progressive learning projects:

- **ex01-next/**: Next.js 15.4.6 web application with React 19.1.0
- **ex02-expo/**: Expo SDK 54 React Native application with New Architecture

Both projects use React Query for data fetching, Axios for HTTP requests, and Back4App/Parse as the backend service.

## Project Selection and Navigation

When working with specific projects, always navigate to the appropriate directory:

```bash
# For Next.js web project
cd ex01-next/

# For Expo React Native project  
cd ex02-expo/
```

## Build, Lint, and Test Commands

### ex01-next (Next.js Project)

```bash
# Development
npm run dev          # Start dev server with Turbopack
npm run build        # Build production version
npm start           # Start production server
npm run lint        # Run ESLint

# Single test execution (when implemented)
npm test -- ComponentName.test.js
npx jest ComponentName.test.js
```

### ex02-expo (React Native Project)

```bash
# Development
npm start           # Start development server
npm run android     # Start for Android
npm run ios        # Start for iOS
npm run web        # Start for web
npm run lint       # Run ESLint with Expo config

# Production builds (requires EAS CLI)
eas build --profile development --platform android
eas build --profile production --platform all

# Single test execution (when implemented)
npm test ComponentName.test.js
```

### Testing Setup (Currently Not Configured)

Neither project has testing configured. When adding tests:

**For ex01-next:**
- Jest + React Testing Library for unit tests
- Playwright or Cypress for E2E tests

**For ex02-expo:**
- Jest + React Native Testing Library
- Detox for E2E testing

## Code Style Guidelines

### File Organization and Naming

**Both Projects:**
- Components: `PascalCase.js/jsx/tsx` (e.g., `CardTask.js`)
- API functions: `camelCase.js` (e.g., `tasks.js`)
- Constants: `UPPER_SNAKE_CASE`

**ex01-next specific:**
- Pages: Use App Router conventions (`page.js`, `layout.js`)
- Styles: `filename.module.css` for component styles

**ex02-expo specific:**
- Routes: File-based routing in `app/` directory
- Route groups: `(groupName)/` for logical grouping
- Dynamic routes: `[id].tsx` for parameter-based routes
- Layouts: `_layout.tsx` for nested layouts

### Import/Export Conventions

**Consistent across both projects:**

```javascript
// 1. React imports first
import React, { useState, useEffect } from 'react';
import { Platform-specific imports } from 'react-native' | 'next';

// 2. Third-party libraries
import { useQuery, useMutation } from '@tanstack/react-query';
import axios from 'axios';

// 3. Internal imports with path alias
import ComponentName from '@/components/ComponentName';
import { apiFunction } from '@/api';

// 4. Relative imports last
import './styles.css'; // Next.js only
```

**Export Patterns:**
- Reusable components: Named exports `export function ComponentName()`
- Page components: Default exports `export default function PageName()`
- API functions: Named exports with descriptive names

### Component Structure Patterns

**Functional Components (Both Projects):**

```javascript
// Page component (default export)
export default function TaskList() {
  // Hooks at the top
  const [loading, setLoading] = useState(false);
  const { data, error } = useQuery({
    queryKey: ['tasks'],
    queryFn: getTasks
  });

  // Event handlers
  const handleClick = () => {
    // Implementation
  };

  // Early returns for conditions
  if (loading) return <LoadingComponent />;
  if (error) return <ErrorComponent error={error} />;

  // Main render
  return (
    <div> {/* or View for React Native */}
      {/* Component content */}
    </div>
  );
}

// Reusable component (named export)
export function CardTask({ task, onUpdate }) {
  return (
    <div style={styles.container}> {/* or View with StyleSheet */}
      {/* Card content */}
    </div>
  );
}
```

### Styling Conventions

**ex01-next (CSS-based):**

```css
/* globals.css - Only for global resets and fonts */
body {
  background-color: blanchedalmond;
}

/* Component.module.css - Scoped component styles */
.container {
  padding: 16px;
  margin-bottom: 10px;
}
```

```javascript
// Component usage
import styles from './Component.module.css';

<div className={styles.container}>Content</div>
```

**ex02-expo (StyleSheet-based):**

```javascript
// Use StyleSheet.create for performance
const styles = StyleSheet.create({
  container: {
    height: 60,
    marginBottom: 10,
    justifyContent: "center",
    backgroundColor: "beige",
  },
  text: {
    fontSize: 16,
    fontWeight: "bold",
  }
});

// Conditional styling with arrays
<View style={[
  styles.container, 
  { backgroundColor: task.done ? "green" : "red" }
]} />
```

### State Management Patterns

**Local State (Both Projects):**
```javascript
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState([]);
```

**Server State with React Query (Both Projects):**
```javascript
// Queries
const { data, isLoading, error } = useQuery({
  queryKey: ['tasks'],
  queryFn: () => getTasks(sessionToken)
});

// Mutations with optimistic updates
const addMutation = useMutation({
  mutationFn: addTask,
  onSuccess: () => {
    queryClient.invalidateQueries(['tasks']);
  },
  onError: (error) => {
    console.log("Error:", error);
    // Handle error appropriately per platform
  }
});
```

**Global State (ex02-expo only - Zustand):**
```javascript
export const useStore = create(
  persist(
    (set) => ({
      user: null,
      count: 0,
      setUser: (user) => set({ user }),
      increment: () => set((state) => ({ count: state.count + 1 })),
    }),
    { name: "store-name", storage: createJSONStorage(() => AsyncStorage) }
  )
);
```

### API Layer Conventions

**Centralized Configuration:**
```javascript
// api/config.js (ex02-expo) or api/index.js (ex01-next)
export const instance = axios.create({
  baseURL: "https://parseapi.back4app.com",
  timeout: 1000,
  headers: {
    "X-Parse-Application-Id": "your-app-id",
    "X-Parse-JavaScript-Key": "your-js-key",
  },
});
```

**Function Naming Convention:**
- GET: `get + Resource` (e.g., `getTasks()`)
- POST: `add + Resource` (e.g., `addTask(task)`)
- PUT: `update + Resource` (e.g., `updateTask(task)`)
- DELETE: `delete + Resource` (e.g., `deleteTask(id)`)

**API Function Structure:**
```javascript
// ex01-next (simple)
export async function getTasks() {
  const response = await instance.get("/classes/Task");
  return response.data;
}

// ex02-expo (with authentication)
export async function getTasks(sessionToken) {
  const response = await instance.get("/classes/Task", {
    headers: { [xParseSessionTokenKey]: sessionToken }
  });
  return response.data;
}
```

### Error Handling Patterns

**ex01-next:**
```javascript
// Simple error display
if (error) {
  return <p>Error: {error.message}</p>;
}
```

**ex02-expo:**
```javascript
// Native alert integration
import { Alert } from 'react-native';

const mutation = useMutation({
  mutationFn: apiFunction,
  onError: (error) => {
    console.log("error", error);
    Alert.alert("Something went wrong", error.message);
  }
});
```

### TypeScript Guidelines

**ex01-next:** Currently uses pure JavaScript with JSConfig path aliases.

**ex02-expo:** Mixed TypeScript/JavaScript:
- Use `.tsx` for React components with complex props
- Use `.ts` for utility functions and API layers  
- Use `.js/.jsx` for simple components
- Configure strict mode in `tsconfig.json`

### Navigation Patterns

**ex01-next (Next.js App Router):**
```javascript
import Link from 'next/link';
import { useRouter } from 'next/navigation';

// Declarative navigation
<Link href="/taskList">Go to Tasks</Link>

// Programmatic navigation
const router = useRouter();
router.push('/taskList');
```

**ex02-expo (Expo Router):**
```javascript
import { useRouter } from 'expo-router';
import { Link } from 'expo-router';

// Declarative navigation
<Link href="/tasks">Go to Tasks</Link>

// Programmatic navigation
const router = useRouter();
router.navigate(`/tasks/${task.objectId}`);
```

## Development Workflow

### For ex01-next:
1. Navigate to `ex01-next/`
2. Run `npm run dev` for development
3. Use browser DevTools for debugging
4. Run `npm run lint` before committing
5. Deploy to Vercel for production

### For ex02-expo:
1. Navigate to `ex02-expo/`
2. Run `expo start` or `npm start`
3. Use Expo DevTools and React DevTools
4. Test on physical devices or simulators
5. Use EAS Build for production builds

## Common Patterns and Best Practices

**Loading States:**
```javascript
// ex01-next
if (isLoading) return <div>Carregando...</div>;

// ex02-expo  
if (isLoading) return <Text>Carregando...</Text>;
```

**Conditional Rendering:**
```javascript
{tasks.length > 0 ? (
  tasks.map(task => <CardTask key={task.objectId} task={task} />)
) : (
  <Text>Nenhuma tarefa encontrada</Text>
)}
```

**Form Handling:**
```javascript
const handleSubmit = (e) => {
  e.preventDefault(); // Web only
  // Handle form submission
};
```

## Performance Guidelines

**Both Projects:**
- Use React.memo() for expensive components
- Implement useMemo/useCallback for expensive calculations
- Leverage React Query caching for API calls

**ex01-next specific:**
- Use Next.js Image component for optimized images
- Implement proper metadata for SEO

**ex02-expo specific:**
- Use FlatList instead of ScrollView for large datasets
- Implement proper keyExtractor for lists
- Use React Native performance monitoring tools

## Security and Environment

- Never commit API keys or secrets
- Use environment variables for configuration
- Validate all user inputs before processing
- Follow Parse/Back4App security guidelines for authentication

## Project-Specific Notes

- **Language:** Portuguese comments and content are acceptable (educational context)
- **Backend:** Uses Parse Server on Back4App - follow Parse API conventions
- **Fonts:** Geist fonts are pre-configured in ex01-next
- **New Architecture:** ex02-expo uses React Native New Architecture (Fabric/TurboModules)

## Git Workflow

- Use descriptive commit messages in English
- Follow conventional commits: `feat:`, `fix:`, `docs:`, `style:`
- Keep commits atomic and focused
- Test changes in appropriate environment before committing

This codebase demonstrates modern React patterns adapted for both web and mobile platforms. When in doubt, refer to the existing code patterns in each project and maintain consistency with the established conventions.