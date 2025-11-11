# Network Layer Implementation

## Overview

This section covers how to implement network calls in React Native, drawing parallels with Retrofit and OkHttp from Android development.

## Fetch API vs Axios vs Retrofit

| Retrofit (Android) | Fetch API (Built-in) | Axios (Library) |
|-------------------|---------------------|-----------------|
| Type-safe interfaces | Manual typing | TypeScript support |
| Built-in interceptors | Manual implementation | Built-in interceptors |
| Automatic serialization | Manual JSON handling | Automatic handling |
| Requires setup | No setup needed | Minimal setup |

## Built-in Fetch API

### Basic Usage

**Kotlin (Retrofit):**
```kotlin
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Int): User

    @POST("users")
    suspend fun createUser(@Body user: User): User
}
```

**TypeScript (Fetch):**
```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

// GET request
const getUser = async (id: number): Promise<User> => {
    try {
        const response = await fetch(`https://api.example.com/users/${id}`);

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error fetching user:', error);
        throw error;
    }
};

// POST request
const createUser = async (user: Omit<User, 'id'>): Promise<User> => {
    try {
        const response = await fetch('https://api.example.com/users', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(user),
        });

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        return await response.json();
    } catch (error) {
        console.error('Error creating user:', error);
        throw error;
    }
};

// PUT request
const updateUser = async (id: number, user: Partial<User>): Promise<User> => {
    const response = await fetch(`https://api.example.com/users/${id}`, {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(user),
    });

    return await response.json();
};

// DELETE request
const deleteUser = async (id: number): Promise<void> => {
    await fetch(`https://api.example.com/users/${id}`, {
        method: 'DELETE',
    });
};
```

## Axios (Recommended)

### Installation

```bash
npm install axios
```

### Basic Setup

```typescript
import axios from 'axios';

const api = axios.create({
    baseURL: 'https://api.example.com',
    timeout: 10000,
    headers: {
        'Content-Type': 'application/json',
    },
});

// GET request
const getUser = async (id: number): Promise<User> => {
    const response = await api.get<User>(`/users/${id}`);
    return response.data;
};

// POST request
const createUser = async (user: Omit<User, 'id'>): Promise<User> => {
    const response = await api.post<User>('/users', user);
    return response.data;
};

// PUT request
const updateUser = async (id: number, user: Partial<User>): Promise<User> => {
    const response = await api.put<User>(`/users/${id}`, user);
    return response.data;
};

// DELETE request
const deleteUser = async (id: number): Promise<void> => {
    await api.delete(`/users/${id}`);
};
```

## API Service Layer

### Organized API Structure

**Android (Retrofit):**
```kotlin
// ApiService.kt
interface ApiService {
    @GET("posts")
    suspend fun getPosts(): List<Post>

    @GET("posts/{id}")
    suspend fun getPost(@Path("id") id: Int): Post
}

// Repository
class PostRepository(private val apiService: ApiService) {
    suspend fun getPosts(): Result<List<Post>> {
        return try {
            Result.success(apiService.getPosts())
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

**TypeScript (Axios):**
```typescript
// api/client.ts
import axios, { AxiosInstance } from 'axios';

class ApiClient {
    private client: AxiosInstance;

    constructor(baseURL: string) {
        this.client = axios.create({
            baseURL,
            timeout: 10000,
            headers: {
                'Content-Type': 'application/json',
            },
        });
    }

    getInstance(): AxiosInstance {
        return this.client;
    }
}

export const apiClient = new ApiClient('https://api.example.com').getInstance();

// api/endpoints/posts.ts
import { apiClient } from '../client';

export interface Post {
    id: number;
    title: string;
    body: string;
    userId: number;
}

export const postsApi = {
    getPosts: async (): Promise<Post[]> => {
        const response = await apiClient.get<Post[]>('/posts');
        return response.data;
    },

    getPost: async (id: number): Promise<Post> => {
        const response = await apiClient.get<Post>(`/posts/${id}`);
        return response.data;
    },

    createPost: async (post: Omit<Post, 'id'>): Promise<Post> => {
        const response = await apiClient.post<Post>('/posts', post);
        return response.data;
    },

    updatePost: async (id: number, post: Partial<Post>): Promise<Post> => {
        const response = await apiClient.put<Post>(`/posts/${id}`, post);
        return response.data;
    },

    deletePost: async (id: number): Promise<void> => {
        await apiClient.delete(`/posts/${id}`);
    },
};

// services/PostService.ts
import { postsApi, Post } from '../api/endpoints/posts';

export class PostService {
    async getPosts(): Promise<Post[]> {
        try {
            return await postsApi.getPosts();
        } catch (error) {
            console.error('Error fetching posts:', error);
            throw error;
        }
    }

    async getPost(id: number): Promise<Post> {
        try {
            return await postsApi.getPost(id);
        } catch (error) {
            console.error('Error fetching post:', error);
            throw error;
        }
    }
}

export const postService = new PostService();
```

## Interceptors (Like OkHttp Interceptors)

### Request Interceptors

**Kotlin (OkHttp):**
```kotlin
class AuthInterceptor(private val tokenProvider: TokenProvider) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val original = chain.request()
        val request = original.newBuilder()
            .header("Authorization", "Bearer ${tokenProvider.getToken()}")
            .build()
        return chain.proceed(request)
    }
}
```

**TypeScript (Axios):**
```typescript
import axios, { AxiosRequestConfig, AxiosError, AxiosResponse } from 'axios';

// Request interceptor (add auth token)
apiClient.interceptors.request.use(
    async (config: AxiosRequestConfig) => {
        // Get token from storage
        const token = await AsyncStorage.getItem('auth_token');

        if (token) {
            config.headers = config.headers || {};
            config.headers.Authorization = `Bearer ${token}`;
        }

        // Log request
        console.log('Request:', config.method?.toUpperCase(), config.url);

        return config;
    },
    (error: AxiosError) => {
        console.error('Request error:', error);
        return Promise.reject(error);
    }
);

// Response interceptor (handle errors globally)
apiClient.interceptors.response.use(
    (response: AxiosResponse) => {
        console.log('Response:', response.status, response.config.url);
        return response;
    },
    async (error: AxiosError) => {
        const originalRequest = error.config;

        // Handle 401 (Unauthorized) - refresh token
        if (error.response?.status === 401 && originalRequest && !originalRequest._retry) {
            originalRequest._retry = true;

            try {
                const refreshToken = await AsyncStorage.getItem('refresh_token');
                const response = await axios.post('/auth/refresh', {
                    refreshToken,
                });

                const { token } = response.data;
                await AsyncStorage.setItem('auth_token', token);

                // Retry original request
                originalRequest.headers = originalRequest.headers || {};
                originalRequest.headers.Authorization = `Bearer ${token}`;
                return apiClient(originalRequest);
            } catch (refreshError) {
                // Redirect to login
                console.error('Token refresh failed:', refreshError);
                // navigation.navigate('Login');
                return Promise.reject(refreshError);
            }
        }

        return Promise.reject(error);
    }
);
```

## Error Handling

### Custom Error Types

```typescript
// types/errors.ts
export class ApiError extends Error {
    constructor(
        public statusCode: number,
        public message: string,
        public data?: any
    ) {
        super(message);
        this.name = 'ApiError';
    }
}

export class NetworkError extends Error {
    constructor(message: string = 'Network error occurred') {
        super(message);
        this.name = 'NetworkError';
    }
}

export class TimeoutError extends Error {
    constructor(message: string = 'Request timeout') {
        super(message);
        this.name = 'TimeoutError';
    }
}

// utils/errorHandler.ts
import { AxiosError } from 'axios';
import { ApiError, NetworkError, TimeoutError } from '../types/errors';

export const handleApiError = (error: unknown): Error => {
    if (axios.isAxiosError(error)) {
        const axiosError = error as AxiosError;

        if (axiosError.code === 'ECONNABORTED') {
            return new TimeoutError();
        }

        if (!axiosError.response) {
            return new NetworkError();
        }

        const { status, data } = axiosError.response;
        return new ApiError(
            status,
            data?.message || 'An error occurred',
            data
        );
    }

    return error instanceof Error ? error : new Error('Unknown error');
};

// Usage in component
const fetchData = async () => {
    try {
        const data = await postService.getPosts();
        setPosts(data);
    } catch (error) {
        const handledError = handleApiError(error);

        if (handledError instanceof ApiError) {
            if (handledError.statusCode === 404) {
                showToast('Posts not found');
            } else if (handledError.statusCode === 500) {
                showToast('Server error');
            }
        } else if (handledError instanceof NetworkError) {
            showToast('Please check your internet connection');
        } else if (handledError instanceof TimeoutError) {
            showToast('Request timeout. Please try again');
        }
    }
};
```

## React Query (TanStack Query) - Advanced Data Fetching

### Installation

```bash
npm install @tanstack/react-query
```

### Setup

```typescript
// App.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
    defaultOptions: {
        queries: {
            retry: 2,
            staleTime: 5 * 60 * 1000, // 5 minutes
            cacheTime: 10 * 60 * 1000, // 10 minutes
        },
    },
});

const App = () => {
    return (
        <QueryClientProvider client={queryClient}>
            {/* Your app components */}
        </QueryClientProvider>
    );
};
```

### Usage in Components

**Android (ViewModel + LiveData):**
```kotlin
class PostViewModel(private val repository: PostRepository) : ViewModel() {
    private val _posts = MutableLiveData<List<Post>>()
    val posts: LiveData<List<Post>> = _posts

    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading

    fun loadPosts() {
        viewModelScope.launch {
            _loading.value = true
            try {
                _posts.value = repository.getPosts()
            } catch (e: Exception) {
                // Handle error
            } finally {
                _loading.value = false
            }
        }
    }
}
```

**TypeScript (React Query):**
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { postsApi, Post } from '../api/endpoints/posts';

// Custom hook for fetching posts
const usePosts = () => {
    return useQuery<Post[], Error>({
        queryKey: ['posts'],
        queryFn: () => postsApi.getPosts(),
    });
};

// Custom hook for fetching single post
const usePost = (id: number) => {
    return useQuery<Post, Error>({
        queryKey: ['posts', id],
        queryFn: () => postsApi.getPost(id),
        enabled: id > 0, // Only fetch if id is valid
    });
};

// Custom hook for creating post
const useCreatePost = () => {
    const queryClient = useQueryClient();

    return useMutation<Post, Error, Omit<Post, 'id'>>({
        mutationFn: (post) => postsApi.createPost(post),
        onSuccess: () => {
            // Invalidate and refetch posts
            queryClient.invalidateQueries({ queryKey: ['posts'] });
        },
    });
};

// Usage in component
const PostsScreen = () => {
    const { data: posts, isLoading, error, refetch } = usePosts();
    const createPost = useCreatePost();

    const handleCreatePost = () => {
        createPost.mutate(
            {
                title: 'New Post',
                body: 'Content',
                userId: 1,
            },
            {
                onSuccess: (data) => {
                    console.log('Post created:', data);
                },
                onError: (error) => {
                    console.error('Error creating post:', error);
                },
            }
        );
    };

    if (isLoading) {
        return <ActivityIndicator />;
    }

    if (error) {
        return <Text>Error: {error.message}</Text>;
    }

    return (
        <FlatList
            data={posts}
            renderItem={({ item }) => <PostItem post={item} />}
            refreshing={isLoading}
            onRefresh={refetch}
        />
    );
};
```

## GraphQL with Apollo Client

### Installation

```bash
npm install @apollo/client graphql
```

### Setup

```typescript
import { ApolloClient, InMemoryCache, ApolloProvider, gql } from '@apollo/client';

const client = new ApolloClient({
    uri: 'https://api.example.com/graphql',
    cache: new InMemoryCache(),
    headers: {
        authorization: `Bearer ${token}`,
    },
});

const App = () => {
    return (
        <ApolloProvider client={client}>
            {/* Your app components */}
        </ApolloProvider>
    );
};
```

### Usage

```typescript
import { useQuery, useMutation, gql } from '@apollo/client';

// Define query
const GET_POSTS = gql`
    query GetPosts {
        posts {
            id
            title
            body
            author {
                name
            }
        }
    }
`;

// Define mutation
const CREATE_POST = gql`
    mutation CreatePost($title: String!, $body: String!) {
        createPost(title: $title, body: $body) {
            id
            title
            body
        }
    }
`;

// Use in component
const PostsScreen = () => {
    const { data, loading, error } = useQuery(GET_POSTS);
    const [createPost] = useMutation(CREATE_POST);

    const handleCreate = async () => {
        try {
            const result = await createPost({
                variables: {
                    title: 'New Post',
                    body: 'Content',
                },
            });
            console.log('Created:', result.data);
        } catch (err) {
            console.error('Error:', err);
        }
    };

    if (loading) return <ActivityIndicator />;
    if (error) return <Text>Error: {error.message}</Text>;

    return (
        <FlatList
            data={data.posts}
            renderItem={({ item }) => <PostItem post={item} />}
        />
    );
};
```

## Offline Support and Caching

### Using React Query for Offline Support

```typescript
import { useQuery } from '@tanstack/react-query';
import { onlineManager } from '@tanstack/react-query';
import NetInfo from '@react-native-community/netinfo';

// Setup network detection
onlineManager.setEventListener((setOnline) => {
    return NetInfo.addEventListener((state) => {
        setOnline(!!state.isConnected);
    });
});

// Query with offline support
const usePosts = () => {
    return useQuery({
        queryKey: ['posts'],
        queryFn: () => postsApi.getPosts(),
        staleTime: 5 * 60 * 1000,
        cacheTime: 24 * 60 * 60 * 1000, // 24 hours
        networkMode: 'offlineFirst', // Try cache first
    });
};
```

### Manual Caching with AsyncStorage

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

const CACHE_KEY = 'posts_cache';
const CACHE_EXPIRY = 60 * 60 * 1000; // 1 hour

interface CacheData<T> {
    data: T;
    timestamp: number;
}

const getCachedData = async <T,>(key: string): Promise<T | null> => {
    try {
        const cached = await AsyncStorage.getItem(key);
        if (!cached) return null;

        const { data, timestamp }: CacheData<T> = JSON.parse(cached);

        // Check if cache is expired
        if (Date.now() - timestamp > CACHE_EXPIRY) {
            await AsyncStorage.removeItem(key);
            return null;
        }

        return data;
    } catch (error) {
        console.error('Error reading cache:', error);
        return null;
    }
};

const setCachedData = async <T,>(key: string, data: T): Promise<void> => {
    try {
        const cacheData: CacheData<T> = {
            data,
            timestamp: Date.now(),
        };
        await AsyncStorage.setItem(key, JSON.stringify(cacheData));
    } catch (error) {
        console.error('Error writing cache:', error);
    }
};

// Usage
const fetchPosts = async (): Promise<Post[]> => {
    // Try cache first
    const cached = await getCachedData<Post[]>(CACHE_KEY);
    if (cached) {
        console.log('Returning cached data');
        return cached;
    }

    // Fetch from API
    const posts = await postsApi.getPosts();

    // Cache the result
    await setCachedData(CACHE_KEY, posts);

    return posts;
};
```

## File Upload

### Using Axios

```typescript
const uploadImage = async (uri: string): Promise<string> => {
    const formData = new FormData();
    formData.append('image', {
        uri,
        type: 'image/jpeg',
        name: 'photo.jpg',
    } as any);

    const response = await apiClient.post<{ url: string }>(
        '/upload',
        formData,
        {
            headers: {
                'Content-Type': 'multipart/form-data',
            },
            onUploadProgress: (progressEvent) => {
                const percentCompleted = Math.round(
                    (progressEvent.loaded * 100) / progressEvent.total
                );
                console.log('Upload progress:', percentCompleted);
            },
        }
    );

    return response.data.url;
};
```

## Best Practices

1. **Create a centralized API client** - single source of configuration
2. **Use interceptors** for common functionality (auth, logging)
3. **Implement proper error handling** - custom error types
4. **Use TypeScript** for type-safe API calls
5. **Implement request/response logging** in development
6. **Handle offline scenarios** gracefully
7. **Cache responses** when appropriate
8. **Use React Query or SWR** for better data management
9. **Implement retry logic** for failed requests
10. **Monitor network performance** and optimize

## Common Pitfalls

1. **Not handling network errors** properly
2. **Blocking UI during network calls** - always show loading states
3. **Not implementing timeout** - requests can hang indefinitely
4. **Exposing API keys** in code - use environment variables
5. **Not validating response data** - can cause runtime errors
6. **Ignoring HTTP status codes** - check beyond just success/failure
7. **Not implementing request cancellation** - memory leaks
8. **Hardcoding URLs** - use configuration files

## Next Steps

Now that you understand networking, let's explore [State Management](./RN-06-state-management.md) to manage application state effectively.
