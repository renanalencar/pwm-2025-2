# AGENTS.md - Development Guidelines

This document provides essential guidelines for coding agents operating in this Next.js codebase. It covers build commands, code style guidelines, and project-specific conventions.

## Project Overview

This is a Next.js 15.4.6 application using React 19.1.0, built for educational purposes (Programação Web e Mobile - UNICAP). The project uses App Router architecture with external Parse/Back4App backend for task management.

## Build/Lint/Test Commands

### Development Commands
```bash
# Start development server with Turbopack
npm run dev

# Build production version
npm run build

# Start production server
npm start

# Run ESLint
npm run lint
```

### Testing Commands
**Note: No testing framework is currently configured.** To add testing:
- Consider Jest + React Testing Library for unit tests
- Consider Playwright or Cypress for E2E tests
- Add test scripts to package.json when implemented

### Single Test Execution
When testing is implemented, use these patterns:
```bash
# Jest (if implemented)
npm test -- ComponentName.test.js
npx jest ComponentName.test.js

# Vitest (if implemented) 
npm test ComponentName.test.js
```

## Code Style Guidelines

### File Organization
```
/app/                 # Next.js App Router pages
/components/          # Reusable React components
/api/                 # API client functions
/public/             # Static assets
/provider.js         # Global providers setup
```

### Import Conventions
```javascript
// External dependencies first
import React from "react";
import { useState } from "react";
import axios from "axios";

// Internal components with @ alias
import ComponentName from "@/components/ComponentName";
import { apiFunction } from "@/api";

// Relative imports last
import "./styles.css";
```

### Component Structure
```javascript
// Use named exports for components
export function ComponentName({ prop1, prop2 }) {
  // Hooks at the top
  const [state, setState] = useState(defaultValue);
  
  // Handler functions
  const handleClick = () => {
    // Implementation
  };

  // Early returns for conditions
  if (loading) return <div>Loading...</div>;

  // Main render
  return (
    <div>
      {/* JSX content */}
    </div>
  );
}
```

### Naming Conventions

**Files:**
- Components: `PascalCase.js` (e.g., `CardTask.js`)
- Pages: `lowercase.js` or `page.js` (App Router)
- Utilities: `camelCase.js`

**Variables/Functions:**
- React components: `PascalCase`
- Functions/variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- Event handlers: `handle + Action` (e.g., `handleClick`)

**Props:**
- Event handlers: `on + Action` (e.g., `onDelete`, `onCheck`)
- Boolean props: `is/has/should + Adjective` (e.g., `isLoading`)

### TypeScript Guidelines
**Note: Project currently uses JavaScript.** When migrating to TypeScript:
- Use `.tsx` for components, `.ts` for utilities
- Define prop types with interfaces
- Use strict TypeScript configuration

### CSS/Styling Conventions
```javascript
// Inline styles (current pattern)
const styles = {
  container: {
    padding: '16px',
    backgroundColor: '#fff'
  }
};

// CSS Modules (when used)
import styles from './Component.module.css';

// Global styles in globals.css only for:
// - CSS resets
// - Font definitions
// - Global variables
```

### State Management Patterns

**Local State:**
```javascript
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
```

**Server State (React Query):**
```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Queries
const { data, isLoading, error } = useQuery({
  queryKey: ['tasks'],
  queryFn: getTasks
});

// Mutations with optimistic updates
const queryClient = useQueryClient();
const mutation = useMutation({
  mutationFn: addTask,
  onSuccess: () => {
    queryClient.invalidateQueries(['tasks']);
  }
});
```

### Error Handling

**API Calls:**
```javascript
try {
  const response = await apiCall();
  return response.data;
} catch (error) {
  console.error('API Error:', error);
  // Handle error appropriately
  throw error;
}
```

**Component Error Boundaries:**
```javascript
// Implement error boundaries for production
// Currently not implemented - add when needed
```

### API Conventions

**File Structure:** Place API functions in `/api/index.js`

**Function Naming:**
- GET: `get + Resource` (e.g., `getTasks()`)
- POST: `add + Resource` (e.g., `addTask(task)`)
- PUT: `update + Resource` (e.g., `updateTask(task)`)
- DELETE: `delete + Resource` (e.g., `deleteTask(id)`)

**Response Handling:**
```javascript
export async function getTasks() {
  const response = await instance.get("/classes/Task");
  return response.data; // Return data directly
}
```

### Performance Guidelines

**React Optimization:**
- Use React.memo for expensive components
- Implement useMemo/useCallback for expensive calculations
- Avoid unnecessary re-renders

**Next.js Optimization:**
- Use Next.js Image component for images
- Implement proper metadata for SEO
- Use font optimization (already implemented)

### Accessibility Guidelines
- Use semantic HTML elements
- Add proper ARIA labels where needed
- Ensure keyboard navigation works
- Test with screen readers

### Security Guidelines
- Never commit API keys (use environment variables)
- Validate all user inputs
- Sanitize data before rendering
- Use HTTPS for production

### Code Comments
```javascript
// Single line for brief explanations
/*
 * Multi-line comments for complex logic
 * or component documentation
 */

/**
 * JSDoc for function documentation
 * @param {string} id - Task identifier
 * @returns {Promise} API response
 */
```

### Git Commit Guidelines
- Use conventional commits: `feat:`, `fix:`, `docs:`, `style:`, `refactor:`
- Keep commits atomic and focused
- Write clear commit messages in English

## Project-Specific Notes

- **Language:** Portuguese comments and content are acceptable (educational project)
- **Backend:** Uses Parse/Back4App - follow their API conventions
- **Routing:** App Router only - avoid Pages Router patterns
- **Fonts:** Geist fonts are pre-configured and optimized
- **Environment:** Development uses Turbopack for faster builds

## Common Patterns

**Loading States:**
```javascript
if (isLoading) return <div>Carregando...</div>;
if (error) return <div>Erro: {error.message}</div>;
```

**Form Handling:**
```javascript
const handleSubmit = (e) => {
  e.preventDefault();
  // Handle form submission
};
```

**Conditional Rendering:**
```javascript
{tasks.length > 0 ? (
  tasks.map(task => <CardTask key={task.objectId} task={task} />)
) : (
  <p>Nenhuma tarefa encontrada</p>
)}
```

This codebase follows modern React/Next.js patterns. When in doubt, refer to Next.js documentation and React best practices. Maintain consistency with existing code patterns.