# Project Structure & Lifecycle

## React Native Project Structure

### Standard Project Anatomy

```
MyApp/
├── android/                    # Android native code
│   ├── app/
│   ├── build.gradle
│   └── settings.gradle
├── ios/                        # iOS native code
│   ├── MyApp/
│   ├── MyApp.xcodeproj/
│   └── Podfile
├── node_modules/               # Dependencies (like build/ in Android)
├── src/                        # Your TypeScript/JavaScript code
│   ├── components/             # Reusable UI components
│   ├── screens/                # Screen components
│   ├── navigation/             # Navigation configuration
│   ├── services/               # API services, business logic
│   ├── store/                  # State management
│   ├── utils/                  # Utility functions
│   ├── hooks/                  # Custom React hooks
│   ├── types/                  # TypeScript type definitions
│   ├── constants/              # App constants
│   └── assets/                 # Images, fonts, etc.
├── __tests__/                  # Test files
├── .eslintrc.js                # ESLint configuration
├── .prettierrc                 # Prettier configuration
├── tsconfig.json               # TypeScript configuration
├── babel.config.js             # Babel configuration
├── metro.config.js             # Metro bundler configuration
├── package.json                # Dependencies and scripts
├── app.json                    # App configuration
├── App.tsx                     # Root component
└── index.js                    # Entry point
```

### Android vs React Native Comparison

| Android/Kotlin | React Native/TypeScript | Purpose |
|----------------|-------------------------|---------|
| `src/main/java/` | `src/` | Source code directory |
| `res/layout/` | Components in `src/` | UI definition |
| `res/values/` | `src/constants/` | Constants and strings |
| `res/drawable/` | `src/assets/` | Images and assets |
| `AndroidManifest.xml` | `app.json` + `android/app/src/main/AndroidManifest.xml` | App configuration |
| `build.gradle` | `package.json` | Dependencies |
| `MainActivity.kt` | `App.tsx` | Entry point |
| `Application` class | `index.js` registration | App initialization |

## Configuration Files

### package.json
Equivalent to `build.gradle` - manages dependencies and scripts.

```json
{
  "name": "MyApp",
  "version": "1.0.0",
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "start": "react-native start",
    "test": "jest",
    "lint": "eslint ."
  },
  "dependencies": {
    "react": "18.2.0",
    "react-native": "0.72.0",
    "@react-navigation/native": "^6.1.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "typescript": "^5.0.0",
    "jest": "^29.5.0"
  }
}
```

### tsconfig.json
TypeScript configuration.

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "commonjs",
    "lib": ["es2017"],
    "jsx": "react-native",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@screens/*": ["src/screens/*"],
      "@services/*": ["src/services/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### metro.config.js
Metro bundler configuration (like Gradle configuration).

```javascript
const {getDefaultConfig} = require('metro-config');

module.exports = (async () => {
  const defaultConfig = await getDefaultConfig();
  return {
    transformer: {
      babelTransformerPath: require.resolve('react-native-svg-transformer'),
    },
    resolver: {
      assetExts: defaultConfig.resolver.assetExts.filter(ext => ext !== 'svg'),
      sourceExts: [...defaultConfig.resolver.sourceExts, 'svg'],
    },
  };
})();
```

### app.json
Basic app configuration.

```json
{
  "name": "MyApp",
  "displayName": "My App",
  "expo": {
    "name": "MyApp",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    }
  }
}
```

## Component Lifecycle

### React Component Lifecycle vs Android Lifecycle

#### Android Activity/Fragment Lifecycle
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Initialize view
    }

    override fun onStart() {
        super.onStart()
        // Component visible
    }

    override fun onResume() {
        super.onResume()
        // Component interactive
    }

    override fun onPause() {
        super.onPause()
        // Component losing focus
    }

    override fun onStop() {
        super.onStop()
        // Component not visible
    }

    override fun onDestroy() {
        super.onDestroy()
        // Cleanup
    }
}
```

#### React Native Functional Component with Hooks

```typescript
import React, { useState, useEffect, useRef } from 'react';
import { AppState, AppStateStatus } from 'react-native';

const MyScreen: React.FC = () => {
    // State initialization (like onCreate)
    const [data, setData] = useState<string[]>([]);
    const [loading, setLoading] = useState(true);
    const appState = useRef(AppState.currentState);

    // Component Mount (onCreate + onStart + onResume)
    useEffect(() => {
        console.log('Component mounted');

        // Initialize data
        loadInitialData();

        // Setup listeners
        const subscription = AppState.addEventListener(
            'change',
            handleAppStateChange
        );

        // Cleanup (onDestroy)
        return () => {
            console.log('Component unmounted');
            subscription.remove();
        };
    }, []); // Empty dependency array = runs once on mount

    // Equivalent to onResume/onPause
    const handleAppStateChange = (nextAppState: AppStateStatus) => {
        if (
            appState.current.match(/inactive|background/) &&
            nextAppState === 'active'
        ) {
            console.log('App has come to the foreground! (onResume)');
            refreshData();
        } else if (
            appState.current === 'active' &&
            nextAppState.match(/inactive|background/)
        ) {
            console.log('App has gone to the background! (onPause)');
            saveState();
        }
        appState.current = nextAppState;
    };

    // Watch specific data changes (like LiveData observer)
    useEffect(() => {
        console.log('Data changed:', data);
        // React to data changes
    }, [data]); // Runs when 'data' changes

    const loadInitialData = async () => {
        try {
            setLoading(true);
            const result = await fetchData();
            setData(result);
        } finally {
            setLoading(false);
        }
    };

    const refreshData = async () => {
        // Refresh when app comes to foreground
    };

    const saveState = () => {
        // Save state when app goes to background
    };

    return (
        // JSX here
    );
};
```

### Lifecycle Comparison Table

| Android Lifecycle | React Native Hook | When It Runs |
|-------------------|-------------------|--------------|
| `onCreate()` | `useEffect(() => {}, [])` | Component first renders |
| `onStart()` | `useEffect(() => {}, [])` | Component first renders |
| `onResume()` | `AppState` listener + `useEffect` | App comes to foreground |
| `onPause()` | `AppState` listener | App goes to background |
| `onStop()` | `AppState` listener | App goes to background |
| `onDestroy()` | `useEffect` cleanup function | Component unmounts |
| LiveData `observe()` | `useEffect(() => {}, [dependency])` | Dependency changes |

### Class Components (Legacy, but still in use)

```typescript
import React, { Component } from 'react';
import { AppState, AppStateStatus } from 'react-native';

interface MyScreenProps {
    userId: string;
}

interface MyScreenState {
    data: string[];
    loading: boolean;
}

class MyScreen extends Component<MyScreenProps, MyScreenState> {
    private appStateSubscription?: any;
    private appState = AppState.currentState;

    // Constructor (like init block in Kotlin)
    constructor(props: MyScreenProps) {
        super(props);
        this.state = {
            data: [],
            loading: true
        };
    }

    // Component Mount (onCreate + onStart + onResume)
    componentDidMount() {
        console.log('Component mounted');
        this.loadInitialData();

        this.appStateSubscription = AppState.addEventListener(
            'change',
            this.handleAppStateChange
        );
    }

    // Component Update (when props or state change)
    componentDidUpdate(
        prevProps: MyScreenProps,
        prevState: MyScreenState
    ) {
        // Like observing LiveData changes
        if (prevProps.userId !== this.props.userId) {
            this.loadInitialData();
        }

        if (prevState.data !== this.state.data) {
            console.log('Data changed');
        }
    }

    // Component Unmount (onDestroy)
    componentWillUnmount() {
        console.log('Component unmounted');
        this.appStateSubscription?.remove();
    }

    handleAppStateChange = (nextAppState: AppStateStatus) => {
        if (
            this.appState.match(/inactive|background/) &&
            nextAppState === 'active'
        ) {
            console.log('App resumed');
            this.refreshData();
        } else if (
            this.appState === 'active' &&
            nextAppState.match(/inactive|background/)
        ) {
            console.log('App paused');
            this.saveState();
        }
        this.appState = nextAppState;
    };

    async loadInitialData() {
        try {
            this.setState({ loading: true });
            const result = await fetchData();
            this.setState({ data: result });
        } finally {
            this.setState({ loading: false });
        }
    }

    async refreshData() {
        // Refresh logic
    }

    saveState() {
        // Save state logic
    }

    render() {
        return (
            // JSX here
        );
    }
}
```

## App State Management

### Handling App Lifecycle Events

```typescript
import { useEffect, useRef } from 'react';
import { AppState, AppStateStatus } from 'react-native';

export const useAppState = (
    onChange: (status: AppStateStatus) => void
) => {
    const appState = useRef(AppState.currentState);

    useEffect(() => {
        const subscription = AppState.addEventListener(
            'change',
            (nextAppState) => {
                if (appState.current !== nextAppState) {
                    appState.current = nextAppState;
                    onChange(nextAppState);
                }
            }
        );

        return () => subscription.remove();
    }, [onChange]);

    return appState.current;
};

// Usage in component
const MyComponent = () => {
    useAppState((status) => {
        if (status === 'active') {
            console.log('App is active');
            // Refresh data, resume tasks
        } else if (status === 'background') {
            console.log('App is in background');
            // Save state, pause tasks
        }
    });

    return (
        // JSX
    );
};
```

### Focus Events (Navigation)

```typescript
import { useFocusEffect } from '@react-navigation/native';
import { useCallback } from 'react';

const MyScreen = () => {
    // Similar to onResume for this screen
    useFocusEffect(
        useCallback(() => {
            console.log('Screen focused');

            // Fetch fresh data
            loadData();

            // Cleanup when screen loses focus
            return () => {
                console.log('Screen unfocused');
            };
        }, [])
    );

    return (
        // JSX
    );
};
```

## Entry Point

### index.js (App Registration)

```javascript
import { AppRegistry } from 'react-native';
import App from './App';
import { name as appName } from './app.json';

// Register the app (like Application class in Android)
AppRegistry.registerComponent(appName, () => App);
```

### App.tsx (Root Component)

**Kotlin (MainActivity):**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

**TypeScript (App.tsx):**
```typescript
import React from 'react';
import { SafeAreaView, StatusBar } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import AppNavigator from './src/navigation/AppNavigator';

const App = () => {
    return (
        <SafeAreaView style={{ flex: 1 }}>
            <StatusBar barStyle="dark-content" />
            <NavigationContainer>
                <AppNavigator />
            </NavigationContainer>
        </SafeAreaView>
    );
};

export default App;
```

## Memory Management

### Android (Manual Management)
```kotlin
class MyActivity : AppCompatActivity() {
    private var listener: SensorListener? = null

    override fun onDestroy() {
        super.onDestroy()
        listener?.unregister()  // Manual cleanup
        listener = null
    }
}
```

### React Native (Automatic with Cleanup Functions)
```typescript
const MyComponent = () => {
    useEffect(() => {
        const subscription = SomeAPI.addListener(() => {
            // Handle event
        });

        // Cleanup automatically called on unmount
        return () => {
            subscription.remove();
        };
    }, []);

    return <View />;
};
```

## Performance Considerations

### React Native Rendering Cycle

1. **State/Props Change** → Component re-renders
2. **Virtual DOM Diff** → Calculate changes
3. **Bridge Communication** → Send to native
4. **Native UI Update** → Render on screen

### Optimization with useMemo and useCallback

```typescript
import React, { useMemo, useCallback, useState } from 'react';

const MyComponent = ({ items }: { items: Item[] }) => {
    const [filter, setFilter] = useState('');

    // Memoize expensive calculations
    const filteredItems = useMemo(() => {
        return items.filter(item =>
            item.name.toLowerCase().includes(filter.toLowerCase())
        );
    }, [items, filter]); // Only recalculate when items or filter changes

    // Memoize callback functions
    const handlePress = useCallback((id: string) => {
        console.log('Pressed:', id);
    }, []); // Function reference stays the same

    return (
        // JSX using filteredItems
    );
};
```

### Component Memoization

```typescript
import React, { memo } from 'react';

// Prevents re-render if props haven't changed
const ExpensiveComponent = memo(({ data }: { data: Data }) => {
    return (
        // Complex UI
    );
}, (prevProps, nextProps) => {
    // Custom comparison function
    // Return true if props are equal (skip re-render)
    return prevProps.data.id === nextProps.data.id;
});
```

## Best Practices

1. **Use functional components** with hooks (modern approach)
2. **Organize by feature**, not by type (screens, components per feature)
3. **Use absolute imports** via path aliases in tsconfig.json
4. **Separate business logic** into custom hooks or services
5. **Keep components small** and focused (single responsibility)
6. **Use TypeScript strictly** for type safety
7. **Clean up subscriptions** in useEffect return functions
8. **Optimize re-renders** with memo, useMemo, useCallback

## Common Pitfalls

1. **Forgetting cleanup** in useEffect can cause memory leaks
2. **Stale closures** in useEffect without proper dependencies
3. **Infinite loops** with incorrect useEffect dependencies
4. **Direct state mutation** - always use setState or state setters
5. **Heavy computations** in render - use useMemo
6. **Creating functions in render** - use useCallback
7. **Not handling loading/error states**

## Project Organization Best Practices

```
src/
├── features/                   # Feature-based organization
│   ├── auth/
│   │   ├── components/
│   │   ├── screens/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types.ts
│   ├── profile/
│   └── home/
├── shared/                     # Shared across features
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── types/
├── navigation/
├── store/                      # Global state
└── theme/                      # Styling constants
```

## Next Steps

Now that you understand the project structure and lifecycle, let's explore [React Native UI Components](./RN-03-ui-components.md) to build user interfaces.
