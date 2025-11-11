# Design Patterns & Architecture

## Overview

This section covers common design patterns and architectural approaches in React Native development, with comparisons to Android/Kotlin patterns you already know.

## 1. Container/Presentational Pattern

Also known as **Smart/Dumb Components** or **Stateful/Stateless Components**.

### Android Equivalent
Similar to separating ViewModels (business logic) from Views (UI).

### Implementation

```typescript
// Presentational Component (Dumb Component)
// Only receives props and renders UI
interface UserCardProps {
    name: string;
    email: string;
    avatarUrl: string;
    onPress: () => void;
}

const UserCard: React.FC<UserCardProps> = ({ name, email, avatarUrl, onPress }) => {
    return (
        <TouchableOpacity style={styles.card} onPress={onPress}>
            <Image source={{ uri: avatarUrl }} style={styles.avatar} />
            <View style={styles.info}>
                <Text style={styles.name}>{name}</Text>
                <Text style={styles.email}>{email}</Text>
            </View>
        </TouchableOpacity>
    );
};

// Container Component (Smart Component)
// Handles logic and state
const UserCardContainer: React.FC<{ userId: string }> = ({ userId }) => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);
    const navigation = useNavigation();

    useEffect(() => {
        loadUser();
    }, [userId]);

    const loadUser = async () => {
        try {
            setLoading(true);
            const userData = await userApi.getUser(userId);
            setUser(userData);
        } finally {
            setLoading(false);
        }
    };

    const handlePress = () => {
        navigation.navigate('UserDetails', { userId });
    };

    if (loading) return <ActivityIndicator />;
    if (!user) return null;

    return (
        <UserCard
            name={user.name}
            email={user.email}
            avatarUrl={user.avatarUrl}
            onPress={handlePress}
        />
    );
};
```

**When to use:**
- Separate business logic from UI
- Make components reusable
- Easier testing
- Clear responsibilities

## 2. Custom Hooks Pattern

Custom hooks encapsulate reusable logic.

### Android Equivalent
Similar to utility classes or ViewModel methods that can be shared.

### Implementation

```typescript
// hooks/useUser.ts
interface UseUserResult {
    user: User | null;
    loading: boolean;
    error: Error | null;
    refetch: () => Promise<void>;
}

export const useUser = (userId: string): UseUserResult => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<Error | null>(null);

    const fetchUser = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const userData = await userApi.getUser(userId);
            setUser(userData);
        } catch (err) {
            setError(err as Error);
        } finally {
            setLoading(false);
        }
    }, [userId]);

    useEffect(() => {
        fetchUser();
    }, [fetchUser]);

    return { user, loading, error, refetch: fetchUser };
};

// hooks/useForm.ts
interface UseFormProps<T> {
    initialValues: T;
    onSubmit: (values: T) => void | Promise<void>;
    validate?: (values: T) => Partial<Record<keyof T, string>>;
}

export const useForm = <T extends Record<string, any>>({
    initialValues,
    onSubmit,
    validate,
}: UseFormProps<T>) => {
    const [values, setValues] = useState<T>(initialValues);
    const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
    const [isSubmitting, setIsSubmitting] = useState(false);

    const handleChange = (field: keyof T) => (value: any) => {
        setValues((prev) => ({ ...prev, [field]: value }));
        // Clear error when user starts typing
        if (errors[field]) {
            setErrors((prev) => {
                const newErrors = { ...prev };
                delete newErrors[field];
                return newErrors;
            });
        }
    };

    const handleSubmit = async () => {
        if (validate) {
            const validationErrors = validate(values);
            if (Object.keys(validationErrors).length > 0) {
                setErrors(validationErrors);
                return;
            }
        }

        setIsSubmitting(true);
        try {
            await onSubmit(values);
        } finally {
            setIsSubmitting(false);
        }
    };

    const resetForm = () => {
        setValues(initialValues);
        setErrors({});
    };

    return {
        values,
        errors,
        isSubmitting,
        handleChange,
        handleSubmit,
        resetForm,
    };
};

// hooks/useDebounce.ts
export const useDebounce = <T,>(value: T, delay: number): T => {
    const [debouncedValue, setDebouncedValue] = useState<T>(value);

    useEffect(() => {
        const handler = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);

        return () => {
            clearTimeout(handler);
        };
    }, [value, delay]);

    return debouncedValue;
};

// Usage in component
const UserProfile = ({ userId }: { userId: string }) => {
    const { user, loading, error, refetch } = useUser(userId);

    if (loading) return <ActivityIndicator />;
    if (error) return <Text>Error: {error.message}</Text>;
    if (!user) return <Text>User not found</Text>;

    return (
        <View>
            <Text>{user.name}</Text>
            <Button title="Refresh" onPress={refetch} />
        </View>
    );
};

const SearchScreen = () => {
    const [searchQuery, setSearchQuery] = useState('');
    const debouncedQuery = useDebounce(searchQuery, 500);

    useEffect(() => {
        if (debouncedQuery) {
            // Perform search
            searchApi.search(debouncedQuery);
        }
    }, [debouncedQuery]);

    return (
        <TextInput
            value={searchQuery}
            onChangeText={setSearchQuery}
            placeholder="Search..."
        />
    );
};
```

## 3. Higher-Order Components (HOC)

A function that takes a component and returns a new component.

### Android Equivalent
Similar to decorator pattern or wrapper classes.

### Implementation

```typescript
// hocs/withAuth.tsx
export const withAuth = <P extends object>(
    Component: React.ComponentType<P>
): React.FC<P> => {
    return (props: P) => {
        const { isAuthenticated, user } = useAuth();
        const navigation = useNavigation();

        useEffect(() => {
            if (!isAuthenticated) {
                navigation.navigate('Login');
            }
        }, [isAuthenticated]);

        if (!isAuthenticated) {
            return <ActivityIndicator />;
        }

        return <Component {...props} />;
    };
};

// hocs/withLoading.tsx
interface WithLoadingProps {
    loading: boolean;
}

export const withLoading = <P extends object>(
    Component: React.ComponentType<P>
): React.FC<P & WithLoadingProps> => {
    return ({ loading, ...props }: WithLoadingProps) => {
        if (loading) {
            return <ActivityIndicator size="large" />;
        }
        return <Component {...(props as P)} />;
    };
};

// Usage
const ProfileScreen = ({ user }: { user: User }) => {
    return (
        <View>
            <Text>{user.name}</Text>
        </View>
    );
};

const ProtectedProfile = withAuth(ProfileScreen);
const ProfileWithLoading = withLoading(ProfileScreen);

// Use in navigation or parent component
<ProtectedProfile user={currentUser} />
<ProfileWithLoading loading={isLoading} user={currentUser} />
```

## 4. Render Props Pattern

Pass a function as a prop to share code between components.

### Implementation

```typescript
// components/DataFetcher.tsx
interface DataFetcherProps<T> {
    url: string;
    children: (data: {
        data: T | null;
        loading: boolean;
        error: Error | null;
        refetch: () => void;
    }) => React.ReactElement;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
    const [data, setData] = useState<T | null>(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<Error | null>(null);

    const fetchData = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const response = await fetch(url);
            const result = await response.json();
            setData(result);
        } catch (err) {
            setError(err as Error);
        } finally {
            setLoading(false);
        }
    }, [url]);

    useEffect(() => {
        fetchData();
    }, [fetchData]);

    return children({ data, loading, error, refetch: fetchData });
}

// Usage
const UserProfile = ({ userId }: { userId: string }) => {
    return (
        <DataFetcher<User> url={`/api/users/${userId}`}>
            {({ data: user, loading, error, refetch }) => {
                if (loading) return <ActivityIndicator />;
                if (error) return <Text>Error: {error.message}</Text>;
                if (!user) return <Text>No user found</Text>;

                return (
                    <View>
                        <Text>{user.name}</Text>
                        <Button title="Refresh" onPress={refetch} />
                    </View>
                );
            }}
        </DataFetcher>
    );
};
```

## 5. Compound Components Pattern

Components that work together to form a complete UI.

### Implementation

```typescript
// components/Accordion.tsx
interface AccordionContextType {
    openItems: Set<string>;
    toggleItem: (id: string) => void;
}

const AccordionContext = createContext<AccordionContextType | undefined>(undefined);

interface AccordionProps {
    children: React.ReactNode;
    allowMultiple?: boolean;
}

export const Accordion: React.FC<AccordionProps> & {
    Item: typeof AccordionItem;
    Header: typeof AccordionHeader;
    Content: typeof AccordionContent;
} = ({ children, allowMultiple = false }) => {
    const [openItems, setOpenItems] = useState<Set<string>>(new Set());

    const toggleItem = (id: string) => {
        setOpenItems((prev) => {
            const next = new Set(allowMultiple ? prev : []);
            if (prev.has(id)) {
                next.delete(id);
            } else {
                next.add(id);
            }
            return next;
        });
    };

    return (
        <AccordionContext.Provider value={{ openItems, toggleItem }}>
            <View>{children}</View>
        </AccordionContext.Provider>
    );
};

const AccordionItem: React.FC<{ id: string; children: React.ReactNode }> = ({
    id,
    children,
}) => {
    return <View style={styles.item}>{children}</View>;
};

const AccordionHeader: React.FC<{ id: string; children: React.ReactNode }> = ({
    id,
    children,
}) => {
    const context = useContext(AccordionContext);
    if (!context) throw new Error('AccordionHeader must be used within Accordion');

    const isOpen = context.openItems.has(id);

    return (
        <TouchableOpacity onPress={() => context.toggleItem(id)}>
            <View style={styles.header}>
                {children}
                <Text>{isOpen ? '▲' : '▼'}</Text>
            </View>
        </TouchableOpacity>
    );
};

const AccordionContent: React.FC<{ id: string; children: React.ReactNode }> = ({
    id,
    children,
}) => {
    const context = useContext(AccordionContext);
    if (!context) throw new Error('AccordionContent must be used within Accordion');

    const isOpen = context.openItems.has(id);

    if (!isOpen) return null;

    return <View style={styles.content}>{children}</View>;
};

Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Content = AccordionContent;

// Usage
const FAQ = () => {
    return (
        <Accordion allowMultiple>
            <Accordion.Item id="1">
                <Accordion.Header id="1">
                    <Text>What is React Native?</Text>
                </Accordion.Header>
                <Accordion.Content id="1">
                    <Text>React Native is a framework...</Text>
                </Accordion.Content>
            </Accordion.Item>

            <Accordion.Item id="2">
                <Accordion.Header id="2">
                    <Text>How does it work?</Text>
                </Accordion.Header>
                <Accordion.Content id="2">
                    <Text>It uses JavaScript...</Text>
                </Accordion.Content>
            </Accordion.Item>
        </Accordion>
    );
};
```

## 6. MVVM Pattern (Model-View-ViewModel)

### Android MVVM

```kotlin
// Model
data class User(val id: String, val name: String)

// ViewModel
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user

    fun loadUser(id: String) {
        viewModelScope.launch {
            _user.value = repository.getUser(id)
        }
    }
}

// View
class UserActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.user.observe(this) { user ->
            // Update UI
        }
        viewModel.loadUser("123")
    }
}
```

### React Native MVVM

```typescript
// Model
export interface User {
    id: string;
    name: string;
    email: string;
}

// ViewModel (Custom Hook)
export const useUserViewModel = (userId: string) => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<Error | null>(null);

    const loadUser = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const userData = await userRepository.getUser(userId);
            setUser(userData);
        } catch (err) {
            setError(err as Error);
        } finally {
            setLoading(false);
        }
    }, [userId]);

    useEffect(() => {
        loadUser();
    }, [loadUser]);

    const updateUser = async (updates: Partial<User>) => {
        if (!user) return;
        try {
            setLoading(true);
            const updated = await userRepository.updateUser(user.id, updates);
            setUser(updated);
        } catch (err) {
            setError(err as Error);
        } finally {
            setLoading(false);
        }
    };

    return {
        user,
        loading,
        error,
        updateUser,
        refresh: loadUser,
    };
};

// View
const UserScreen: React.FC<{ userId: string }> = ({ userId }) => {
    const { user, loading, error, updateUser, refresh } = useUserViewModel(userId);

    if (loading) return <ActivityIndicator />;
    if (error) return <Text>Error: {error.message}</Text>;
    if (!user) return <Text>User not found</Text>;

    return (
        <View>
            <Text>{user.name}</Text>
            <Text>{user.email}</Text>
            <Button title="Refresh" onPress={refresh} />
        </View>
    );
};
```

## 7. Clean Architecture

### Layer Structure

```
src/
├── domain/                     # Business logic (pure TypeScript)
│   ├── entities/
│   │   └── User.ts
│   ├── usecases/
│   │   ├── GetUser.ts
│   │   └── UpdateUser.ts
│   └── repositories/
│       └── UserRepository.ts   # Interface
├── data/                       # Data layer
│   ├── repositories/
│   │   └── UserRepositoryImpl.ts
│   ├── datasources/
│   │   ├── UserRemoteDataSource.ts
│   │   └── UserLocalDataSource.ts
│   └── models/
│       └── UserDto.ts
└── presentation/               # UI layer
    ├── screens/
    ├── components/
    └── viewmodels/
```

### Implementation

```typescript
// domain/entities/User.ts
export interface User {
    id: string;
    name: string;
    email: string;
}

// domain/repositories/UserRepository.ts
export interface UserRepository {
    getUser(id: string): Promise<User>;
    updateUser(id: string, updates: Partial<User>): Promise<User>;
}

// domain/usecases/GetUser.ts
export class GetUserUseCase {
    constructor(private repository: UserRepository) {}

    async execute(userId: string): Promise<User> {
        return await this.repository.getUser(userId);
    }
}

// data/models/UserDto.ts
export interface UserDto {
    id: string;
    name: string;
    email: string;
    created_at: string; // API specific field
}

// data/datasources/UserRemoteDataSource.ts
export class UserRemoteDataSource {
    async fetchUser(id: string): Promise<UserDto> {
        const response = await apiClient.get<UserDto>(`/users/${id}`);
        return response.data;
    }

    async updateUser(id: string, updates: Partial<UserDto>): Promise<UserDto> {
        const response = await apiClient.put<UserDto>(`/users/${id}`, updates);
        return response.data;
    }
}

// data/datasources/UserLocalDataSource.ts
export class UserLocalDataSource {
    async getUser(id: string): Promise<UserDto | null> {
        const data = await AsyncStorage.getItem(`user_${id}`);
        return data ? JSON.parse(data) : null;
    }

    async saveUser(user: UserDto): Promise<void> {
        await AsyncStorage.setItem(`user_${user.id}`, JSON.stringify(user));
    }
}

// data/repositories/UserRepositoryImpl.ts
export class UserRepositoryImpl implements UserRepository {
    constructor(
        private remoteDataSource: UserRemoteDataSource,
        private localDataSource: UserLocalDataSource
    ) {}

    async getUser(id: string): Promise<User> {
        try {
            // Try cache first
            const cached = await this.localDataSource.getUser(id);
            if (cached) {
                return this.mapDtoToEntity(cached);
            }

            // Fetch from API
            const dto = await this.remoteDataSource.fetchUser(id);

            // Cache it
            await this.localDataSource.saveUser(dto);

            return this.mapDtoToEntity(dto);
        } catch (error) {
            // Fallback to cache if network fails
            const cached = await this.localDataSource.getUser(id);
            if (cached) {
                return this.mapDtoToEntity(cached);
            }
            throw error;
        }
    }

    async updateUser(id: string, updates: Partial<User>): Promise<User> {
        const dto = await this.remoteDataSource.updateUser(id, updates);
        await this.localDataSource.saveUser(dto);
        return this.mapDtoToEntity(dto);
    }

    private mapDtoToEntity(dto: UserDto): User {
        return {
            id: dto.id,
            name: dto.name,
            email: dto.email,
        };
    }
}

// presentation/viewmodels/useUserViewModel.ts
export const useUserViewModel = (userId: string) => {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<Error | null>(null);

    // Dependency injection
    const getUserUseCase = useMemo(
        () =>
            new GetUserUseCase(
                new UserRepositoryImpl(
                    new UserRemoteDataSource(),
                    new UserLocalDataSource()
                )
            ),
        []
    );

    const loadUser = useCallback(async () => {
        try {
            setLoading(true);
            setError(null);
            const userData = await getUserUseCase.execute(userId);
            setUser(userData);
        } catch (err) {
            setError(err as Error);
        } finally {
            setLoading(false);
        }
    }, [userId, getUserUseCase]);

    useEffect(() => {
        loadUser();
    }, [loadUser]);

    return { user, loading, error, refresh: loadUser };
};
```

## 8. Repository Pattern

Abstraction layer between data sources and business logic.

```typescript
// repositories/PostRepository.ts
export interface PostRepository {
    getPosts(): Promise<Post[]>;
    getPost(id: number): Promise<Post>;
    createPost(post: CreatePostDto): Promise<Post>;
    updatePost(id: number, updates: Partial<Post>): Promise<Post>;
    deletePost(id: number): Promise<void>;
}

export class PostRepositoryImpl implements PostRepository {
    constructor(
        private api: PostApi,
        private cache: PostCache
    ) {}

    async getPosts(): Promise<Post[]> {
        // Try cache first
        const cached = await this.cache.getPosts();
        if (cached && !this.cache.isExpired('posts')) {
            return cached;
        }

        // Fetch from API
        const posts = await this.api.getPosts();

        // Update cache
        await this.cache.setPosts(posts);

        return posts;
    }

    async getPost(id: number): Promise<Post> {
        const cached = await this.cache.getPost(id);
        if (cached && !this.cache.isExpired(`post_${id}`)) {
            return cached;
        }

        const post = await this.api.getPost(id);
        await this.cache.setPost(post);
        return post;
    }

    async createPost(post: CreatePostDto): Promise<Post> {
        const created = await this.api.createPost(post);
        await this.cache.invalidate('posts');
        return created;
    }

    async updatePost(id: number, updates: Partial<Post>): Promise<Post> {
        const updated = await this.api.updatePost(id, updates);
        await this.cache.setPost(updated);
        await this.cache.invalidate('posts');
        return updated;
    }

    async deletePost(id: number): Promise<void> {
        await this.api.deletePost(id);
        await this.cache.removePost(id);
        await this.cache.invalidate('posts');
    }
}
```

## 9. Dependency Injection

### Simple DI with Context

```typescript
// di/ServiceContainer.ts
export class ServiceContainer {
    private static instance: ServiceContainer;

    public userRepository: UserRepository;
    public postRepository: PostRepository;
    public authService: AuthService;

    private constructor() {
        // Initialize services
        const apiClient = new ApiClient(API_BASE_URL);

        this.userRepository = new UserRepositoryImpl(
            new UserRemoteDataSource(apiClient),
            new UserLocalDataSource()
        );

        this.postRepository = new PostRepositoryImpl(
            new PostApi(apiClient),
            new PostCache()
        );

        this.authService = new AuthService(
            apiClient,
            this.userRepository
        );
    }

    static getInstance(): ServiceContainer {
        if (!ServiceContainer.instance) {
            ServiceContainer.instance = new ServiceContainer();
        }
        return ServiceContainer.instance;
    }
}

// di/ServiceContext.tsx
const ServiceContext = createContext<ServiceContainer | null>(null);

export const ServiceProvider: React.FC<{ children: React.ReactNode }> = ({
    children,
}) => {
    const services = useMemo(() => ServiceContainer.getInstance(), []);

    return (
        <ServiceContext.Provider value={services}>
            {children}
        </ServiceContext.Provider>
    );
};

export const useServices = (): ServiceContainer => {
    const context = useContext(ServiceContext);
    if (!context) {
        throw new Error('useServices must be used within ServiceProvider');
    }
    return context;
};

// Usage
const UserScreen = ({ userId }: { userId: string }) => {
    const { userRepository } = useServices();
    const [user, setUser] = useState<User | null>(null);

    useEffect(() => {
        userRepository.getUser(userId).then(setUser);
    }, [userId, userRepository]);

    return user ? <Text>{user.name}</Text> : <ActivityIndicator />;
};
```

## Best Practices

1. **Choose patterns based on complexity** - don't over-engineer
2. **Be consistent** - use same patterns across the project
3. **Separate concerns** - clear boundaries between layers
4. **Make it testable** - pure functions, dependency injection
5. **Document architecture** - help future developers
6. **Use TypeScript** - type safety prevents many issues
7. **Keep components small** - single responsibility
8. **Prefer composition over inheritance**
9. **Use custom hooks** for reusable logic
10. **Consider team size and experience**

## Common Pitfalls

1. **Over-engineering small apps** - keep it simple
2. **Mixing concerns** - business logic in UI components
3. **Tight coupling** - hard to test and maintain
4. **Ignoring TypeScript** - loses type safety
5. **Not thinking about testing** - hard to test later
6. **Premature optimization** - optimize when needed
7. **Inconsistent patterns** - confuses developers

## Next Steps

Now that you understand design patterns, let's explore [Advanced Topics & Best Practices](./RN-08-advanced-best-practices.md) to master React Native development.
