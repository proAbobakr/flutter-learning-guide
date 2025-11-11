# State Management Deep Dive

## Overview

State management is crucial for building scalable React Native applications. This guide compares different solutions and helps you choose the right one for your needs.

## Android State Management vs React Native

| Android/Kotlin | React Native |
|----------------|--------------|
| ViewModel + LiveData | useState + Context |
| StateFlow | useState + useEffect |
| SharedFlow | EventEmitter/Context |
| Room Database | AsyncStorage + State |
| Dagger/Hilt DI | Context API/DI libraries |
| SavedStateHandle | AsyncStorage |

## 1. Local Component State (useState)

### Basic Usage

**Kotlin (ViewModel):**
```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableLiveData(0)
    val count: LiveData<Int> = _count

    fun increment() {
        _count.value = (_count.value ?: 0) + 1
    }
}
```

**TypeScript (useState):**
```typescript
import { useState } from 'react';

const CounterScreen = () => {
    const [count, setCount] = useState(0);

    const increment = () => {
        setCount(count + 1);
        // or using functional update
        setCount(prev => prev + 1);
    };

    return (
        <View>
            <Text>Count: {count}</Text>
            <Button title="Increment" onPress={increment} />
        </View>
    );
};
```

### useReducer (For Complex State)

```typescript
import { useReducer } from 'react';

interface State {
    count: number;
    step: number;
}

type Action =
    | { type: 'increment' }
    | { type: 'decrement' }
    | { type: 'setStep'; payload: number }
    | { type: 'reset' };

const initialState: State = {
    count: 0,
    step: 1,
};

const reducer = (state: State, action: Action): State => {
    switch (action.type) {
        case 'increment':
            return { ...state, count: state.count + state.step };
        case 'decrement':
            return { ...state, count: state.count - state.step };
        case 'setStep':
            return { ...state, step: action.payload };
        case 'reset':
            return initialState;
        default:
            return state;
    }
};

const CounterScreen = () => {
    const [state, dispatch] = useReducer(reducer, initialState);

    return (
        <View>
            <Text>Count: {state.count}</Text>
            <Button
                title="Increment"
                onPress={() => dispatch({ type: 'increment' })}
            />
            <Button
                title="Decrement"
                onPress={() => dispatch({ type: 'decrement' })}
            />
            <Button
                title="Reset"
                onPress={() => dispatch({ type: 'reset' })}
            />
        </View>
    );
};
```

**When to use:**
- Simple, component-specific state
- Form inputs
- UI toggles
- Local component data

## 2. Context API

### Basic Setup

```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

// Define context shape
interface AuthContextType {
    user: User | null;
    login: (email: string, password: string) => Promise<void>;
    logout: () => void;
    isAuthenticated: boolean;
}

// Create context
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider component
export const AuthProvider = ({ children }: { children: ReactNode }) => {
    const [user, setUser] = useState<User | null>(null);

    const login = async (email: string, password: string) => {
        try {
            const response = await authApi.login(email, password);
            setUser(response.user);
            await AsyncStorage.setItem('user', JSON.stringify(response.user));
        } catch (error) {
            throw error;
        }
    };

    const logout = async () => {
        setUser(null);
        await AsyncStorage.removeItem('user');
    };

    const value: AuthContextType = {
        user,
        login,
        logout,
        isAuthenticated: user !== null,
    };

    return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

// Custom hook for using context
export const useAuth = (): AuthContextType => {
    const context = useContext(AuthContext);
    if (context === undefined) {
        throw new Error('useAuth must be used within AuthProvider');
    }
    return context;
};

// App setup
const App = () => {
    return (
        <AuthProvider>
            <NavigationContainer>
                {/* Your app */}
            </NavigationContainer>
        </AuthProvider>
    );
};

// Usage in component
const ProfileScreen = () => {
    const { user, logout, isAuthenticated } = useAuth();

    if (!isAuthenticated) {
        return <LoginScreen />;
    }

    return (
        <View>
            <Text>Welcome, {user?.name}</Text>
            <Button title="Logout" onPress={logout} />
        </View>
    );
};
```

### Multiple Contexts

```typescript
// Theme Context
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Language Context
const LanguageContext = createContext<LanguageContextType | undefined>(undefined);

// Settings Context
const SettingsContext = createContext<SettingsContextType | undefined>(undefined);

// Combine providers
const AppProviders = ({ children }: { children: ReactNode }) => {
    return (
        <AuthProvider>
            <ThemeProvider>
                <LanguageProvider>
                    <SettingsProvider>
                        {children}
                    </SettingsProvider>
                </LanguageProvider>
            </ThemeProvider>
        </AuthProvider>
    );
};
```

**When to use:**
- Sharing data across many components
- Theme management
- User authentication state
- Language/localization
- Small to medium-sized apps

**Limitations:**
- Performance issues with frequent updates
- Can cause unnecessary re-renders
- Becomes complex with multiple contexts
- No built-in devtools

## 3. Redux & Redux Toolkit

### Installation

```bash
npm install @reduxjs/toolkit react-redux
```

### Setup with Redux Toolkit

```typescript
// store/slices/counterSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
    value: number;
    step: number;
}

const initialState: CounterState = {
    value: 0,
    step: 1,
};

const counterSlice = createSlice({
    name: 'counter',
    initialState,
    reducers: {
        increment: (state) => {
            state.value += state.step; // Immer allows mutation
        },
        decrement: (state) => {
            state.value -= state.step;
        },
        incrementByAmount: (state, action: PayloadAction<number>) => {
            state.value += action.payload;
        },
        setStep: (state, action: PayloadAction<number>) => {
            state.step = action.payload;
        },
        reset: (state) => {
            state.value = 0;
        },
    },
});

export const { increment, decrement, incrementByAmount, setStep, reset } = counterSlice.actions;
export default counterSlice.reducer;

// store/slices/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
    id: string;
    name: string;
    email: string;
}

interface UserState {
    user: User | null;
    loading: boolean;
    error: string | null;
}

const initialState: UserState = {
    user: null,
    loading: false,
    error: null,
};

// Async thunk (like coroutine in Kotlin)
export const fetchUser = createAsyncThunk(
    'user/fetchUser',
    async (userId: string, { rejectWithValue }) => {
        try {
            const user = await userApi.getUser(userId);
            return user;
        } catch (error: any) {
            return rejectWithValue(error.message);
        }
    }
);

export const loginUser = createAsyncThunk(
    'user/login',
    async (credentials: { email: string; password: string }) => {
        const response = await authApi.login(credentials);
        return response.user;
    }
);

const userSlice = createSlice({
    name: 'user',
    initialState,
    reducers: {
        logout: (state) => {
            state.user = null;
        },
        updateUser: (state, action: PayloadAction<Partial<User>>) => {
            if (state.user) {
                state.user = { ...state.user, ...action.payload };
            }
        },
    },
    extraReducers: (builder) => {
        builder
            // fetchUser
            .addCase(fetchUser.pending, (state) => {
                state.loading = true;
                state.error = null;
            })
            .addCase(fetchUser.fulfilled, (state, action) => {
                state.loading = false;
                state.user = action.payload;
            })
            .addCase(fetchUser.rejected, (state, action) => {
                state.loading = false;
                state.error = action.payload as string;
            })
            // loginUser
            .addCase(loginUser.fulfilled, (state, action) => {
                state.user = action.payload;
            });
    },
});

export const { logout, updateUser } = userSlice.actions;
export default userSlice.reducer;

// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './slices/counterSlice';
import userReducer from './slices/userSlice';

export const store = configureStore({
    reducer: {
        counter: counterReducer,
        user: userReducer,
    },
    middleware: (getDefaultMiddleware) =>
        getDefaultMiddleware({
            serializableCheck: false, // For AsyncStorage
        }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// hooks/redux.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from '../store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// App.tsx
import { Provider } from 'react-redux';
import { store } from './store';

const App = () => {
    return (
        <Provider store={store}>
            <NavigationContainer>
                {/* Your app */}
            </NavigationContainer>
        </Provider>
    );
};

// Usage in component
import { useAppSelector, useAppDispatch } from '../hooks/redux';
import { increment, decrement, reset } from '../store/slices/counterSlice';
import { fetchUser, logout } from '../store/slices/userSlice';

const MyScreen = () => {
    const dispatch = useAppDispatch();
    const count = useAppSelector((state) => state.counter.value);
    const { user, loading, error } = useAppSelector((state) => state.user);

    useEffect(() => {
        dispatch(fetchUser('123'));
    }, []);

    return (
        <View>
            <Text>Count: {count}</Text>
            <Button title="Increment" onPress={() => dispatch(increment())} />
            <Button title="Reset" onPress={() => dispatch(reset())} />

            {loading && <ActivityIndicator />}
            {user && <Text>User: {user.name}</Text>}
            <Button title="Logout" onPress={() => dispatch(logout())} />
        </View>
    );
};
```

**When to use:**
- Large applications
- Complex state logic
- Need for time-travel debugging
- Multiple developers
- Predictable state updates

**Pros:**
- Predictable state management
- Excellent DevTools
- Large ecosystem
- Well-documented
- Time-travel debugging

**Cons:**
- Boilerplate (reduced with Redux Toolkit)
- Learning curve
- Overkill for small apps

## 4. MobX

### Installation

```bash
npm install mobx mobx-react-lite
```

### Setup

```typescript
// stores/CounterStore.ts
import { makeAutoObservable } from 'mobx';

class CounterStore {
    count = 0;
    step = 1;

    constructor() {
        makeAutoObservable(this);
    }

    increment() {
        this.count += this.step;
    }

    decrement() {
        this.count -= this.step;
    }

    setStep(step: number) {
        this.step = step;
    }

    reset() {
        this.count = 0;
    }

    get doubleCount() {
        return this.count * 2;
    }
}

export const counterStore = new CounterStore();

// stores/UserStore.ts
import { makeAutoObservable, runInAction } from 'mobx';

class UserStore {
    user: User | null = null;
    loading = false;
    error: string | null = null;

    constructor() {
        makeAutoObservable(this);
    }

    async fetchUser(userId: string) {
        this.loading = true;
        this.error = null;

        try {
            const user = await userApi.getUser(userId);
            runInAction(() => {
                this.user = user;
                this.loading = false;
            });
        } catch (error: any) {
            runInAction(() => {
                this.error = error.message;
                this.loading = false;
            });
        }
    }

    async login(email: string, password: string) {
        this.loading = true;
        try {
            const response = await authApi.login(email, password);
            runInAction(() => {
                this.user = response.user;
                this.loading = false;
            });
        } catch (error: any) {
            runInAction(() => {
                this.error = error.message;
                this.loading = false;
            });
        }
    }

    logout() {
        this.user = null;
    }

    get isAuthenticated() {
        return this.user !== null;
    }
}

export const userStore = new UserStore();

// stores/index.ts
import { createContext, useContext } from 'react';
import { counterStore } from './CounterStore';
import { userStore } from './UserStore';

export const stores = {
    counterStore,
    userStore,
};

const StoreContext = createContext(stores);

export const useStores = () => useContext(StoreContext);

// Usage in component
import { observer } from 'mobx-react-lite';
import { useStores } from '../stores';

const CounterScreen = observer(() => {
    const { counterStore } = useStores();

    return (
        <View>
            <Text>Count: {counterStore.count}</Text>
            <Text>Double: {counterStore.doubleCount}</Text>
            <Button title="Increment" onPress={() => counterStore.increment()} />
            <Button title="Reset" onPress={() => counterStore.reset()} />
        </View>
    );
});

const UserScreen = observer(() => {
    const { userStore } = useStores();

    useEffect(() => {
        userStore.fetchUser('123');
    }, []);

    if (userStore.loading) {
        return <ActivityIndicator />;
    }

    return (
        <View>
            {userStore.user && <Text>User: {userStore.user.name}</Text>}
            <Button title="Logout" onPress={() => userStore.logout()} />
        </View>
    );
});
```

**When to use:**
- Object-oriented approach preferred
- Need automatic reactivity
- Coming from Android/Kotlin background (similar to observable patterns)
- Rapid development

**Pros:**
- Less boilerplate
- Automatic reactivity
- Easy to learn
- OOP-friendly
- Similar to Kotlin's observable patterns

**Cons:**
- Less predictable than Redux
- Smaller community
- Easier to create memory leaks
- Magic can be confusing

## 5. Zustand (Lightweight Alternative)

### Installation

```bash
npm install zustand
```

### Setup

```typescript
// stores/useCounterStore.ts
import { create } from 'zustand';

interface CounterState {
    count: number;
    step: number;
    increment: () => void;
    decrement: () => void;
    setStep: (step: number) => void;
    reset: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
    count: 0,
    step: 1,
    increment: () => set((state) => ({ count: state.count + state.step })),
    decrement: () => set((state) => ({ count: state.count - state.step })),
    setStep: (step) => set({ step }),
    reset: () => set({ count: 0 }),
}));

// stores/useUserStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface UserState {
    user: User | null;
    loading: boolean;
    error: string | null;
    fetchUser: (userId: string) => Promise<void>;
    login: (email: string, password: string) => Promise<void>;
    logout: () => void;
}

export const useUserStore = create<UserState>()(
    persist(
        (set) => ({
            user: null,
            loading: false,
            error: null,

            fetchUser: async (userId) => {
                set({ loading: true, error: null });
                try {
                    const user = await userApi.getUser(userId);
                    set({ user, loading: false });
                } catch (error: any) {
                    set({ error: error.message, loading: false });
                }
            },

            login: async (email, password) => {
                set({ loading: true, error: null });
                try {
                    const response = await authApi.login(email, password);
                    set({ user: response.user, loading: false });
                } catch (error: any) {
                    set({ error: error.message, loading: false });
                }
            },

            logout: () => set({ user: null }),
        }),
        {
            name: 'user-storage',
            storage: createJSONStorage(() => AsyncStorage),
        }
    )
);

// Usage in component
const CounterScreen = () => {
    const count = useCounterStore((state) => state.count);
    const increment = useCounterStore((state) => state.increment);
    const reset = useCounterStore((state) => state.reset);

    // Or get everything
    const { count, increment, reset } = useCounterStore();

    return (
        <View>
            <Text>Count: {count}</Text>
            <Button title="Increment" onPress={increment} />
            <Button title="Reset" onPress={reset} />
        </View>
    );
};

const UserScreen = () => {
    const { user, loading, fetchUser, logout } = useUserStore();

    useEffect(() => {
        fetchUser('123');
    }, []);

    if (loading) return <ActivityIndicator />;

    return (
        <View>
            {user && <Text>User: {user.name}</Text>}
            <Button title="Logout" onPress={logout} />
        </View>
    );
};
```

**When to use:**
- Want simplicity with less boilerplate
- Small to medium-sized apps
- Don't need Redux complexity
- Need persistence easily

**Pros:**
- Minimal boilerplate
- Small bundle size (~1KB)
- Easy to learn
- Built-in persistence
- No providers needed
- TypeScript-friendly

**Cons:**
- Smaller ecosystem
- Less tooling
- Not as battle-tested as Redux

## 6. Recoil

### Installation

```bash
npm install recoil
```

### Setup

```typescript
// App.tsx
import { RecoilRoot } from 'recoil';

const App = () => {
    return (
        <RecoilRoot>
            <NavigationContainer>
                {/* Your app */}
            </NavigationContainer>
        </RecoilRoot>
    );
};

// atoms/counterAtom.ts
import { atom, selector } from 'recoil';

export const countState = atom({
    key: 'countState',
    default: 0,
});

export const stepState = atom({
    key: 'stepState',
    default: 1,
});

export const doubleCountState = selector({
    key: 'doubleCountState',
    get: ({ get }) => {
        const count = get(countState);
        return count * 2;
    },
});

// atoms/userAtom.ts
import { atom, selector, selectorFamily } from 'recoil';

export const userState = atom<User | null>({
    key: 'userState',
    default: null,
});

export const userIdState = atom<string | null>({
    key: 'userIdState',
    default: null,
});

export const userQuery = selectorFamily({
    key: 'userQuery',
    get: (userId: string) => async () => {
        const user = await userApi.getUser(userId);
        return user;
    },
});

export const isAuthenticatedState = selector({
    key: 'isAuthenticatedState',
    get: ({ get }) => {
        const user = get(userState);
        return user !== null;
    },
});

// Usage in component
import { useRecoilState, useRecoilValue, useSetRecoilState } from 'recoil';
import { countState, stepState, doubleCountState } from '../atoms/counterAtom';

const CounterScreen = () => {
    const [count, setCount] = useRecoilState(countState);
    const [step, setStep] = useRecoilState(stepState);
    const doubleCount = useRecoilValue(doubleCountState);

    const increment = () => setCount(count + step);
    const reset = () => setCount(0);

    return (
        <View>
            <Text>Count: {count}</Text>
            <Text>Double: {doubleCount}</Text>
            <Button title="Increment" onPress={increment} />
            <Button title="Reset" onPress={reset} />
        </View>
    );
};

// Async example
import { useRecoilValueLoadable } from 'recoil';
import { userQuery } from '../atoms/userAtom';

const UserScreen = ({ userId }: { userId: string }) => {
    const userLoadable = useRecoilValueLoadable(userQuery(userId));

    switch (userLoadable.state) {
        case 'loading':
            return <ActivityIndicator />;
        case 'hasError':
            return <Text>Error: {userLoadable.contents}</Text>;
        case 'hasValue':
            return <Text>User: {userLoadable.contents.name}</Text>;
    }
};
```

**When to use:**
- Need fine-grained reactivity
- Working with graphs of state
- React-first approach
- Async state management

**Pros:**
- React-like API
- Fine-grained updates
- Built-in async support
- Derived state
- Easy to learn for React developers

**Cons:**
- Experimental (though stable)
- Smaller community
- Facebook-backed but not as popular
- Less documentation

## 7. Jotai (Atomic State)

### Installation

```bash
npm install jotai
```

### Setup

```typescript
// atoms/counterAtom.ts
import { atom } from 'jotai';

export const countAtom = atom(0);
export const stepAtom = atom(1);

export const doubleCountAtom = atom((get) => get(countAtom) * 2);

export const incrementAtom = atom(
    null,
    (get, set) => {
        set(countAtom, get(countAtom) + get(stepAtom));
    }
);

// Usage in component
import { useAtom, useAtomValue, useSetAtom } from 'jotai';
import { countAtom, stepAtom, doubleCountAtom, incrementAtom } from '../atoms/counterAtom';

const CounterScreen = () => {
    const [count, setCount] = useAtom(countAtom);
    const doubleCount = useAtomValue(doubleCountAtom);
    const increment = useSetAtom(incrementAtom);

    return (
        <View>
            <Text>Count: {count}</Text>
            <Text>Double: {doubleCount}</Text>
            <Button title="Increment" onPress={() => increment()} />
            <Button title="Reset" onPress={() => setCount(0)} />
        </View>
    );
};
```

**When to use:**
- Prefer atomic approach
- Need simplicity
- TypeScript-first
- Small bundle size

**Pros:**
- Minimal API
- TypeScript-first
- No provider needed
- Small bundle size
- Simple async support

**Cons:**
- Newer library
- Smaller community
- Less documentation

## Comparison Matrix

| Feature | Context API | Redux Toolkit | MobX | Zustand | Recoil | Jotai |
|---------|------------|---------------|------|---------|--------|-------|
| **Bundle Size** | 0KB (built-in) | ~9KB | ~16KB | ~1KB | ~14KB | ~3KB |
| **Learning Curve** | Easy | Medium | Easy | Easy | Easy | Easy |
| **Boilerplate** | Low | Medium | Low | Very Low | Low | Very Low |
| **DevTools** | No | Excellent | Good | Basic | Good | Basic |
| **TypeScript** | Good | Excellent | Good | Excellent | Good | Excellent |
| **Performance** | Can be slow | Excellent | Excellent | Excellent | Excellent | Excellent |
| **Async Support** | Manual | Built-in (thunks) | Built-in | Manual | Built-in | Built-in |
| **Middleware** | No | Yes | Yes | Yes | No | No |
| **Persistence** | Manual | Manual | Manual | Built-in | Manual | Manual |
| **Time Travel** | No | Yes | No | No | No | No |
| **Community** | Large | Largest | Large | Growing | Medium | Growing |

## When to Use Each Solution

### Context API
✅ Theme management
✅ User authentication (simple)
✅ Language/localization
✅ Small apps
❌ Frequent updates
❌ Complex state logic
❌ Large apps

### Redux Toolkit
✅ Large applications
✅ Complex state logic
✅ Team projects
✅ Need debugging tools
✅ Predictable state updates
❌ Simple apps (overkill)
❌ Rapid prototyping

### MobX
✅ OOP developers
✅ Coming from Kotlin/Android
✅ Rapid development
✅ Automatic reactivity
❌ Need strict predictability
❌ Junior developers

### Zustand
✅ Modern apps
✅ Less boilerplate
✅ Need simplicity
✅ Small to medium apps
❌ Need Redux ecosystem
❌ Time-travel debugging

### Recoil
✅ React-focused teams
✅ Complex async state
✅ Derived state
✅ Graph-like state
❌ Production stability concerns

### Jotai
✅ Atomic state approach
✅ TypeScript projects
✅ Minimal bundle size
✅ Simple async
❌ Need large ecosystem

## Migration Strategies

### From Context API to Redux

```typescript
// Before (Context)
const { user, setUser } = useAuth();

// After (Redux)
const user = useAppSelector(state => state.auth.user);
const dispatch = useAppDispatch();
dispatch(setUser(newUser));
```

### From Redux to Zustand

```typescript
// Before (Redux)
const dispatch = useAppDispatch();
const count = useAppSelector(state => state.counter.count);
dispatch(increment());

// After (Zustand)
const { count, increment } = useCounterStore();
increment();
```

## Best Practices

1. **Choose based on app complexity** - don't over-engineer
2. **Consider team experience** - learning curve matters
3. **Think about scalability** - will it grow?
4. **Performance first** - avoid unnecessary re-renders
5. **Use TypeScript** - type safety is crucial
6. **Separate concerns** - UI state vs server state
7. **Consider React Query/SWR** for server state
8. **Persist when needed** - user preferences, auth tokens
9. **Test your state logic** - pure functions are testable
10. **Document state structure** - help future developers

## Next Steps

Now that you understand state management, let's explore [Design Patterns & Architecture](./RN-07-design-patterns-architecture.md) to structure your code effectively.
