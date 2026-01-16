# AGENTS.md - Development Guidelines for ex02-expo

## Project Overview
This is an **Expo React Native** project using Expo SDK 54 with the New Architecture (Fabric/TurboModules) enabled. It uses file-based routing with Expo Router v6, React Query for data fetching, Zustand for state management, and Back4App as the backend.

## Build, Lint, and Test Commands

### Development Commands
```bash
# Start development server (choose platform when prompted)
npm start
# or
expo start

# Platform-specific development
npm run android     # Start for Android
npm run ios        # Start for iOS  
npm run web        # Start for web

# Reset project structure (removes example code)
npm run reset-project
```

### Linting and Code Quality
```bash
# Run ESLint (using Expo's flat config)
npm run lint

# Auto-fix linting issues
npx expo lint --fix
```

### Build Commands
```bash
# Development builds (requires EAS CLI)
eas build --profile development --platform ios
eas build --profile development --platform android

# Production builds
eas build --profile production --platform all
```

### Testing Commands
**Note**: No testing framework is currently configured. To add testing:
```bash
# Install Jest and React Native Testing Library (recommended)
npm install --save-dev jest @testing-library/react-native @testing-library/jest-native

# Add to package.json scripts:
# "test": "jest",
# "test:watch": "jest --watch"
```

## Code Style Guidelines

### File Structure and Naming
- **Routes**: Use file-based routing in `app/` directory
  - Route groups: `(groupName)/` for authentication flows
  - Dynamic routes: `[id].tsx` or `[...slug].tsx`
  - Layouts: `_layout.tsx` for nested layouts
- **Components**: PascalCase files in `components/` (e.g., `CardTask.js`)
- **API**: Lowercase files in `api/` (e.g., `tasks.js`, `users.js`)
- **Types**: Use `.tsx` for React components, `.ts` for utilities

### Import Conventions
```javascript
// 1. React and React Native imports first
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';

// 2. Third-party libraries
import { useQuery } from '@tanstack/react-query';
import { useRouter } from 'expo-router';

// 3. Local imports using path alias (@/)
import { useStore } from '@/zustand';
import { getTasks } from '@/api';
import { CardTask } from '@/components/CardTask';
```

### TypeScript Guidelines
- Use strict mode (`"strict": true` in tsconfig.json)
- Define proper types for API responses and component props
- Use path mapping: `@/*` points to project root
- Prefer interface over type for object definitions

### Component Patterns

#### Functional Components (Preferred)
```javascript
// Export as default for route components
export default function TaskList() {
  const { user } = useStore();
  // Component logic
  return <View>...</View>;
}

// Named exports for reusable components
export function CardTask({ task, onNavigate }) {
  return <View>...</View>;
}
```

#### Styling Patterns
```javascript
// Use StyleSheet.create for performance
const styles = StyleSheet.create({
  container: {
    height: 60,
    marginBottom: 10,
    justifyContent: "center",
  },
});

// Conditional styling
<View style={[styles.container, { backgroundColor: task.done ? "green" : "red" }]} />
```

### State Management

#### Zustand Store Pattern
```javascript
export const useStore = create(
  persist(
    (set) => ({
      // State
      user: null,
      count: 0,
      
      // Actions
      setUser: (user) => set({ user }),
      clearUser: () => set({ user: null }),
      inc: () => set((state) => ({ count: state.count + 1 })),
    }),
    { name: "store-name", storage: createJSONStorage(() => AsyncStorage) }
  )
);
```

#### React Query Patterns
```javascript
// Query pattern
const { data, isFetching, error } = useQuery({
  queryKey: ["todos"],
  queryFn: () => getTasks(user?.sessionToken),
});

// Mutation pattern
const addMutation = useMutation({
  mutationFn: addTask,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["todos"] });
  },
  onError: (error) => {
    Alert.alert("Something went wrong", error.message);
  },
});
```

### API Layer Conventions

#### Axios Configuration
```javascript
// Use centralized instance with base configuration
export const instance = axios.create({
  baseURL: "https://parseapi.back4app.com",
  timeout: 1000,
  headers: {
    "X-Parse-Application-Id": "your-app-id",
    "X-Parse-JavaScript-Key": "your-js-key",
  },
});
```

#### API Function Patterns
```javascript
// Export async functions that accept necessary parameters
export const getTasks = async (sessionToken) => {
  const response = await instance.get("/classes/Task", {
    headers: { [xParseSessionTokenKey]: sessionToken },
  });
  return response.data;
};
```

### Error Handling

#### React Query Error Handling
```javascript
// Handle loading and error states in components
if (isFetching) return <Text>Loading...</Text>;
if (error) return <Text>Error: {error.message}</Text>;
if (!data) return <Text>No data available</Text>;
```

#### Async Function Error Handling
```javascript
// Use try-catch for async operations
const addMutation = useMutation({
  mutationFn: addTask,
  onError: (error) => {
    console.log("error", error);
    Alert.alert("Something went wrong", error.message);
  },
});
```

### Navigation Patterns

#### Expo Router Navigation
```javascript
// Use useRouter for programmatic navigation
const router = useRouter();
router.navigate(`/tasks/${task.objectId}`);

// Use useNavigation for screen options
const navigation = useNavigation();
useEffect(() => {
  navigation.setOptions({ title: `Tarefas` });
}, [navigation]);
```

### Environment Configuration
- Use `app.json` for Expo configuration
- Environment variables go in EAS configuration
- Use `expo-constants` to access build-time configuration

### Performance Considerations
- Use `FlatList` for large lists instead of `ScrollView`
- Implement proper `keyExtractor` for FlatList performance
- Use `React.memo()` for expensive components
- Leverage React Query caching for API calls

## Development Workflow
1. Run `expo start` to begin development
2. Use platform-specific commands for testing on devices/simulators
3. Commit changes with descriptive messages
4. Run `npm run lint` before pushing changes
5. Use EAS Build for production builds and OTA updates

## Architecture Notes
- **New Architecture**: Fabric and TurboModules enabled for better performance
- **File-based Routing**: Routes automatically generated from `app/` directory structure
- **Global State**: Zustand with AsyncStorage persistence for offline capabilities
- **Data Fetching**: React Query for caching, background updates, and optimistic updates
- **Backend**: Parse Server on Back4App for authentication and data storage