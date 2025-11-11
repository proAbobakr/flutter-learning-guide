# Advanced Topics & Best Practices

## Performance Optimization

### 1. Preventing Unnecessary Re-renders

#### React.memo

```typescript
// Without memo - re-renders on every parent render
const ExpensiveComponent = ({ data }: { data: Data }) => {
    console.log('Rendering...');
    return <ComplexUI data={data} />;
};

// With memo - only re-renders when data changes
const ExpensiveComponent = React.memo(({ data }: { data: Data }) => {
    console.log('Rendering...');
    return <ComplexUI data={data} />;
});

// With custom comparison
const ExpensiveComponent = React.memo(
    ({ data }: { data: Data }) => {
        return <ComplexUI data={data} />;
    },
    (prevProps, nextProps) => {
        // Return true if props are equal (skip re-render)
        return prevProps.data.id === nextProps.data.id;
    }
);
```

#### useMemo - Memoize Expensive Calculations

```typescript
const UserList = ({ users, filter }: { users: User[]; filter: string }) => {
    // Without useMemo - recalculates on every render
    const filteredUsers = users.filter(user =>
        user.name.toLowerCase().includes(filter.toLowerCase())
    );

    // With useMemo - only recalculates when dependencies change
    const filteredUsers = useMemo(() => {
        console.log('Filtering users...');
        return users.filter(user =>
            user.name.toLowerCase().includes(filter.toLowerCase())
        );
    }, [users, filter]);

    return (
        <FlatList
            data={filteredUsers}
            renderItem={({ item }) => <UserItem user={item} />}
        />
    );
};
```

#### useCallback - Memoize Functions

```typescript
const ParentComponent = () => {
    const [count, setCount] = useState(0);

    // Without useCallback - new function on every render
    const handlePress = () => {
        console.log('Pressed');
    };

    // With useCallback - same function reference
    const handlePress = useCallback(() => {
        console.log('Pressed');
    }, []); // Empty deps - never changes

    const handlePressWithDeps = useCallback(() => {
        console.log('Count:', count);
    }, [count]); // Re-creates when count changes

    return <ChildComponent onPress={handlePress} />;
};
```

### 2. FlatList Optimization

```typescript
interface Item {
    id: string;
    title: string;
    description: string;
}

const OptimizedList: React.FC<{ data: Item[] }> = ({ data }) => {
    // Memoize renderItem
    const renderItem = useCallback(({ item }: { item: Item }) => {
        return <ItemComponent item={item} />;
    }, []);

    // Memoize keyExtractor
    const keyExtractor = useCallback((item: Item) => item.id, []);

    // Memoize getItemLayout for fixed-height items (huge performance boost)
    const getItemLayout = useCallback(
        (data: any, index: number) => ({
            length: ITEM_HEIGHT,
            offset: ITEM_HEIGHT * index,
            index,
        }),
        []
    );

    return (
        <FlatList
            data={data}
            renderItem={renderItem}
            keyExtractor={keyExtractor}
            getItemLayout={getItemLayout}
            // Performance props
            maxToRenderPerBatch={10}
            updateCellsBatchingPeriod={50}
            initialNumToRender={10}
            windowSize={5}
            removeClippedSubviews={true}
            // Memory optimization
            onEndReachedThreshold={0.5}
        />
    );
};

// Memoized item component
const ItemComponent = React.memo(({ item }: { item: Item }) => {
    return (
        <View style={styles.item}>
            <Text>{item.title}</Text>
            <Text>{item.description}</Text>
        </View>
    );
});
```

### 3. Image Optimization

```typescript
import FastImage from 'react-native-fast-image';

const OptimizedImage = ({ uri }: { uri: string }) => {
    return (
        <FastImage
            style={styles.image}
            source={{
                uri,
                priority: FastImage.priority.normal,
                cache: FastImage.cacheControl.immutable,
            }}
            resizeMode={FastImage.resizeMode.cover}
        />
    );
};

// For local images, use require for bundling
const LocalImage = () => {
    return (
        <Image
            source={require('../assets/image.png')}
            style={styles.image}
        />
    );
};

// Lazy loading images
const LazyImage = ({ uri }: { uri: string }) => {
    const [loaded, setLoaded] = useState(false);

    return (
        <View>
            {!loaded && <ActivityIndicator />}
            <Image
                source={{ uri }}
                style={styles.image}
                onLoad={() => setLoaded(true)}
            />
        </View>
    );
};
```

### 4. Bundle Size Optimization

```typescript
// Use dynamic imports for large libraries
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

const App = () => {
    return (
        <Suspense fallback={<ActivityIndicator />}>
            <HeavyComponent />
        </Suspense>
    );
};

// Analyze bundle size
// Run: npx react-native-bundle-visualizer
```

## Testing Strategies

### 1. Unit Testing with Jest

```typescript
// utils/validation.ts
export const validateEmail = (email: string): boolean => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
};

export const validatePassword = (password: string): boolean => {
    return password.length >= 8;
};

// utils/validation.test.ts
import { validateEmail, validatePassword } from './validation';

describe('Validation Utils', () => {
    describe('validateEmail', () => {
        it('should return true for valid email', () => {
            expect(validateEmail('test@example.com')).toBe(true);
        });

        it('should return false for invalid email', () => {
            expect(validateEmail('invalid-email')).toBe(false);
            expect(validateEmail('test@')).toBe(false);
            expect(validateEmail('@example.com')).toBe(false);
        });
    });

    describe('validatePassword', () => {
        it('should return true for password with 8+ characters', () => {
            expect(validatePassword('password123')).toBe(true);
        });

        it('should return false for password with less than 8 characters', () => {
            expect(validatePassword('pass')).toBe(false);
        });
    });
});
```

### 2. Component Testing

```typescript
// components/Button.test.tsx
import { render, fireEvent } from '@testing-library/react-native';
import { Button } from './Button';

describe('Button Component', () => {
    it('renders correctly', () => {
        const { getByText } = render(<Button title="Press Me" />);
        expect(getByText('Press Me')).toBeTruthy();
    });

    it('calls onPress when pressed', () => {
        const onPressMock = jest.fn();
        const { getByText } = render(
            <Button title="Press Me" onPress={onPressMock} />
        );

        fireEvent.press(getByText('Press Me'));
        expect(onPressMock).toHaveBeenCalledTimes(1);
    });

    it('is disabled when loading', () => {
        const { getByText } = render(
            <Button title="Press Me" loading={true} />
        );

        const button = getByText('Press Me');
        expect(button.props.accessibilityState.disabled).toBe(true);
    });
});
```

### 3. Hook Testing

```typescript
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from './useCounter';

describe('useCounter', () => {
    it('should initialize with default value', () => {
        const { result } = renderHook(() => useCounter());
        expect(result.current.count).toBe(0);
    });

    it('should increment counter', () => {
        const { result } = renderHook(() => useCounter());

        act(() => {
            result.current.increment();
        });

        expect(result.current.count).toBe(1);
    });

    it('should decrement counter', () => {
        const { result } = renderHook(() => useCounter(5));

        act(() => {
            result.current.decrement();
        });

        expect(result.current.count).toBe(4);
    });
});
```

### 4. Integration Testing

```typescript
// screens/LoginScreen.test.tsx
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { LoginScreen } from './LoginScreen';
import * as authApi from '../api/auth';

jest.mock('../api/auth');

describe('LoginScreen', () => {
    it('should login successfully with valid credentials', async () => {
        const mockLogin = jest.spyOn(authApi, 'login').mockResolvedValue({
            user: { id: '1', name: 'John' },
            token: 'fake-token',
        });

        const { getByPlaceholderText, getByText } = render(<LoginScreen />);

        const emailInput = getByPlaceholderText('Email');
        const passwordInput = getByPlaceholderText('Password');
        const submitButton = getByText('Login');

        fireEvent.changeText(emailInput, 'test@example.com');
        fireEvent.changeText(passwordInput, 'password123');
        fireEvent.press(submitButton);

        await waitFor(() => {
            expect(mockLogin).toHaveBeenCalledWith(
                'test@example.com',
                'password123'
            );
        });
    });

    it('should show error message for invalid credentials', async () => {
        jest.spyOn(authApi, 'login').mockRejectedValue(
            new Error('Invalid credentials')
        );

        const { getByPlaceholderText, getByText, findByText } = render(
            <LoginScreen />
        );

        fireEvent.changeText(getByPlaceholderText('Email'), 'test@example.com');
        fireEvent.changeText(getByPlaceholderText('Password'), 'wrong');
        fireEvent.press(getByText('Login'));

        const errorMessage = await findByText('Invalid credentials');
        expect(errorMessage).toBeTruthy();
    });
});
```

## Debugging Techniques

### 1. React DevTools

```bash
# Install
npm install -g react-devtools

# Run
react-devtools

# In your app (index.js)
if (__DEV__) {
    require('react-devtools');
}
```

### 2. Flipper Integration

```typescript
// Enable in your app
import { useEffect } from 'react';

if (__DEV__) {
    require('react-native-flipper').addPlugin(/* your plugin */);
}

// Network debugging
import NetworkLogger from 'react-native-network-logger';

<NetworkLogger />
```

### 3. Console Debugging

```typescript
// Better logging
const logger = {
    log: (message: string, data?: any) => {
        if (__DEV__) {
            console.log(`[LOG] ${message}`, data);
        }
    },
    error: (message: string, error?: any) => {
        if (__DEV__) {
            console.error(`[ERROR] ${message}`, error);
        }
    },
    warn: (message: string, data?: any) => {
        if (__DEV__) {
            console.warn(`[WARN] ${message}`, data);
        }
    },
};

// Usage
logger.log('User logged in', { userId: '123' });
logger.error('Failed to fetch data', error);
```

### 4. Performance Monitoring

```typescript
import { InteractionManager } from 'react-native';

// Measure screen render time
const MyScreen = () => {
    useEffect(() => {
        const startTime = Date.now();

        InteractionManager.runAfterInteractions(() => {
            const endTime = Date.now();
            console.log(`Screen rendered in ${endTime - startTime}ms`);
        });
    }, []);

    return <View />;
};

// Measure function execution time
const measurePerformance = (name: string, fn: () => void) => {
    const start = Date.now();
    fn();
    const end = Date.now();
    console.log(`${name} took ${end - start}ms`);
};
```

## Code Organization Best Practices

### 1. Feature-Based Structure

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── RegisterForm.tsx
│   │   ├── screens/
│   │   │   ├── LoginScreen.tsx
│   │   │   └── RegisterScreen.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── store/
│   │   │   └── authSlice.ts
│   │   ├── api/
│   │   │   └── authApi.ts
│   │   └── types.ts
│   ├── profile/
│   └── settings/
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── types/
├── navigation/
├── theme/
└── config/
```

### 2. Naming Conventions

```typescript
// Components - PascalCase
const UserProfile = () => {};

// Hooks - camelCase with 'use' prefix
const useUser = () => {};

// Constants - UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';

// Types/Interfaces - PascalCase
interface User {}
type UserRole = 'admin' | 'user';

// Files
// - Components: UserProfile.tsx
// - Screens: LoginScreen.tsx
// - Hooks: useAuth.ts
// - Utils: validation.ts
// - Types: user.types.ts
```

### 3. Import Organization

```typescript
// 1. External libraries
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';

// 2. Internal modules (absolute imports)
import { Button } from '@/components/Button';
import { useAuth } from '@/hooks/useAuth';
import { User } from '@/types';

// 3. Relative imports
import { validateEmail } from '../utils/validation';
import { styles } from './styles';
```

## Security Best Practices

### 1. Secure Storage

```typescript
import * as Keychain from 'react-native-keychain';

// Store sensitive data
export const storeCredentials = async (
    username: string,
    password: string
) => {
    try {
        await Keychain.setGenericPassword(username, password);
    } catch (error) {
        console.error('Error storing credentials:', error);
    }
};

// Retrieve sensitive data
export const getCredentials = async () => {
    try {
        const credentials = await Keychain.getGenericPassword();
        if (credentials) {
            return {
                username: credentials.username,
                password: credentials.password,
            };
        }
        return null;
    } catch (error) {
        console.error('Error retrieving credentials:', error);
        return null;
    }
};

// Remove credentials
export const removeCredentials = async () => {
    try {
        await Keychain.resetGenericPassword();
    } catch (error) {
        console.error('Error removing credentials:', error);
    }
};
```

### 2. Environment Variables

```typescript
// .env file
API_BASE_URL=https://api.example.com
API_KEY=your_api_key_here

// React Native Config
import Config from 'react-native-config';

export const API_BASE_URL = Config.API_BASE_URL;
export const API_KEY = Config.API_KEY;

// NEVER commit .env to git
// Add to .gitignore
```

### 3. API Security

```typescript
// Don't expose API keys in code
// ❌ Bad
const API_KEY = 'sk_live_abc123';

// ✅ Good - use environment variables
const API_KEY = Config.API_KEY;

// Use HTTPS only
const apiClient = axios.create({
    baseURL: 'https://api.example.com', // Not http://
});

// Implement certificate pinning for sensitive apps
import { fetch } from 'react-native-ssl-pinning';

fetch('https://api.example.com', {
    method: 'GET',
    sslPinning: {
        certs: ['cert1', 'cert2'],
    },
});
```

## Memory Management

### 1. Cleanup in useEffect

```typescript
const MyComponent = () => {
    useEffect(() => {
        // Setup
        const subscription = eventEmitter.addListener('event', handler);
        const interval = setInterval(() => {}, 1000);

        // Cleanup
        return () => {
            subscription.remove();
            clearInterval(interval);
        };
    }, []);

    return <View />;
};
```

### 2. Avoid Memory Leaks

```typescript
// ❌ Bad - can cause memory leak
const MyComponent = () => {
    useEffect(() => {
        fetchData().then(data => {
            // Component might be unmounted
            setState(data);
        });
    }, []);
};

// ✅ Good - check if mounted
const MyComponent = () => {
    useEffect(() => {
        let isMounted = true;

        fetchData().then(data => {
            if (isMounted) {
                setState(data);
            }
        });

        return () => {
            isMounted = false;
        };
    }, []);
};

// ✅ Better - use AbortController
const MyComponent = () => {
    useEffect(() => {
        const abortController = new AbortController();

        fetch(url, { signal: abortController.signal })
            .then(response => response.json())
            .then(data => setState(data))
            .catch(error => {
                if (error.name !== 'AbortError') {
                    console.error(error);
                }
            });

        return () => {
            abortController.abort();
        };
    }, []);
};
```

## Accessibility

```typescript
import { AccessibilityInfo } from 'react-native';

const AccessibleButton = () => {
    return (
        <TouchableOpacity
            accessible={true}
            accessibilityLabel="Submit button"
            accessibilityHint="Double tap to submit the form"
            accessibilityRole="button"
            accessibilityState={{ disabled: false }}
        >
            <Text>Submit</Text>
        </TouchableOpacity>
    );
};

// Check if screen reader is enabled
const checkScreenReader = async () => {
    const screenReaderEnabled = await AccessibilityInfo.isScreenReaderEnabled();
    console.log('Screen reader enabled:', screenReaderEnabled);
};
```

## Common Pitfalls

1. **Not handling loading states** - always show feedback
2. **Forgetting cleanup** - memory leaks
3. **Not optimizing lists** - performance issues
4. **Hardcoding dimensions** - doesn't scale
5. **Not handling errors** - app crashes
6. **Ignoring TypeScript warnings** - runtime errors
7. **Not testing on both platforms** - platform-specific bugs
8. **Committing sensitive data** - security risk
9. **Not using absolute imports** - messy imports
10. **Over-optimizing prematurely** - wasted time

## Production Checklist

- [ ] Environment variables configured
- [ ] API keys secured
- [ ] Error tracking implemented (Sentry)
- [ ] Analytics integrated
- [ ] Performance monitoring enabled
- [ ] All images optimized
- [ ] Bundle size analyzed
- [ ] Tests passing
- [ ] Both platforms tested
- [ ] Offline functionality handled
- [ ] Loading states implemented
- [ ] Error boundaries added
- [ ] Accessibility tested
- [ ] Memory leaks fixed
- [ ] Security review completed
- [ ] Documentation updated

## Conclusion

Congratulations! You've completed the React Native and TypeScript learning guide for Kotlin Android developers. You should now have a solid understanding of:

1. TypeScript fundamentals and how they compare to Kotlin
2. React Native project structure and component lifecycle
3. Building UIs with React Native components
4. Popular libraries and the React Native ecosystem
5. Implementing network layers and API calls
6. Various state management solutions and when to use them
7. Design patterns and architectural approaches
8. Performance optimization and best practices

## Continue Learning

- Build real projects
- Contribute to open source
- Read official documentation
- Join React Native communities
- Follow best practices
- Stay updated with new releases

**Happy coding!** 🚀
