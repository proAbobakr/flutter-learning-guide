# Section 6: Network Layer

## Introduction

Building a robust network layer is crucial for any app. This section covers HTTP clients, REST API integration, serialization, error handling, and best practices.

## HTTP Client Options

### 1. http Package (Basic)

**Android Equivalent:** HttpURLConnection

```yaml
dependencies:
  http: ^1.1.2
```

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class ApiService {
  static const String baseUrl = 'https://api.example.com';

  Future<User> getUser(int id) async {
    final response = await http.get(
      Uri.parse('$baseUrl/users/$id'),
      headers: {'Authorization': 'Bearer $token'},
    );

    if (response.statusCode == 200) {
      return User.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to load user');
    }
  }

  Future<User> createUser(User user) async {
    final response = await http.post(
      Uri.parse('$baseUrl/users'),
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $token',
      },
      body: jsonEncode(user.toJson()),
    );

    if (response.statusCode == 201) {
      return User.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to create user');
    }
  }
}
```

### 2. Dio Package (Recommended)

**Android Equivalent:** Retrofit + OkHttp

```yaml
dependencies:
  dio: ^5.4.0
```

## Complete Network Layer Architecture

### Project Structure

```
lib/
├── data/
│   ├── models/
│   │   ├── user_model.dart
│   │   └── response_wrapper.dart
│   ├── api/
│   │   ├── api_client.dart
│   │   ├── api_endpoints.dart
│   │   ├── api_interceptors.dart
│   │   └── api_error.dart
│   └── repositories/
│       └── user_repository.dart
```

### Step 1: Define Models

```dart
// data/models/user_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user_model.freezed.dart';
part 'user_model.g.dart';

@freezed
class User with _$User {
  const factory User({
    required int id,
    required String name,
    required String email,
    @JsonKey(name: 'phone_number') String? phoneNumber,
    @JsonKey(name: 'created_at') DateTime? createdAt,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// Generic response wrapper
@freezed
class ApiResponse<T> with _$ApiResponse<T> {
  const factory ApiResponse({
    required bool success,
    String? message,
    T? data,
    @JsonKey(name: 'error_code') String? errorCode,
  }) = _ApiResponse<T>;

  factory ApiResponse.fromJson(
    Map<String, dynamic> json,
    T Function(Object? json) fromJsonT,
  ) =>
      _$ApiResponseFromJson(json, fromJsonT);
}
```

### Step 2: API Endpoints

```dart
// data/api/api_endpoints.dart
class ApiEndpoints {
  static const String baseUrl = 'https://api.example.com/v1';

  // Auth
  static const String login = '/auth/login';
  static const String register = '/auth/register';
  static const String logout = '/auth/logout';

  // Users
  static const String users = '/users';
  static String userById(int id) => '/users/$id';
  static String userPosts(int id) => '/users/$id/posts';

  // Posts
  static const String posts = '/posts';
  static String postById(int id) => '/posts/$id';
}
```

### Step 3: Custom Exceptions

```dart
// data/api/api_error.dart
class ApiError implements Exception {
  final String message;
  final int? statusCode;
  final String? errorCode;
  final dynamic data;

  ApiError({
    required this.message,
    this.statusCode,
    this.errorCode,
    this.data,
  });

  @override
  String toString() => message;
}

class NetworkError extends ApiError {
  NetworkError({String? message})
      : super(
          message: message ?? 'No internet connection',
          statusCode: -1,
        );
}

class TimeoutError extends ApiError {
  TimeoutError({String? message})
      : super(
          message: message ?? 'Request timeout',
          statusCode: -2,
        );
}

class UnauthorizedError extends ApiError {
  UnauthorizedError({String? message})
      : super(
          message: message ?? 'Unauthorized',
          statusCode: 401,
        );
}

class ServerError extends ApiError {
  ServerError({String? message})
      : super(
          message: message ?? 'Server error',
          statusCode: 500,
        );
}
```

### Step 4: Interceptors

```dart
// data/api/api_interceptors.dart
import 'package:dio/dio.dart';

class AuthInterceptor extends Interceptor {
  final TokenManager _tokenManager;

  AuthInterceptor(this._tokenManager);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await _tokenManager.getToken();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    return handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    if (err.response?.statusCode == 401) {
      // Try to refresh token
      try {
        final newToken = await _tokenManager.refreshToken();
        if (newToken != null) {
          // Retry request with new token
          final opts = err.requestOptions;
          opts.headers['Authorization'] = 'Bearer $newToken';

          final dio = Dio();
          final response = await dio.fetch(opts);
          return handler.resolve(response);
        }
      } catch (e) {
        // Refresh failed, logout user
        await _tokenManager.clearToken();
        // Navigate to login
      }
    }
    return handler.next(err);
  }
}

class LoggingInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    print('REQUEST[${options.method}] => PATH: ${options.path}');
    print('Headers: ${options.headers}');
    print('Data: ${options.data}');
    return super.onRequest(options, handler);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    print(
      'RESPONSE[${response.statusCode}] => PATH: ${response.requestOptions.path}',
    );
    print('Data: ${response.data}');
    return super.onResponse(response, handler);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    print(
      'ERROR[${err.response?.statusCode}] => PATH: ${err.requestOptions.path}',
    );
    print('Message: ${err.message}');
    return super.onError(err, handler);
  }
}

class RetryInterceptor extends Interceptor {
  final int maxRetries;
  final Duration retryDelay;

  RetryInterceptor({
    this.maxRetries = 3,
    this.retryDelay = const Duration(seconds: 1),
  });

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    final extra = err.requestOptions.extra;
    final retries = extra['retries'] ?? 0;

    if (retries < maxRetries && _shouldRetry(err)) {
      extra['retries'] = retries + 1;
      await Future.delayed(retryDelay * (retries + 1));

      try {
        final response = await Dio().fetch(err.requestOptions);
        return handler.resolve(response);
      } catch (e) {
        return super.onError(err, handler);
      }
    }

    return super.onError(err, handler);
  }

  bool _shouldRetry(DioException err) {
    return err.type == DioExceptionType.connectionTimeout ||
        err.type == DioExceptionType.receiveTimeout ||
        err.type == DioExceptionType.sendTimeout ||
        (err.response?.statusCode ?? 0) >= 500;
  }
}
```

### Step 5: API Client

```dart
// data/api/api_client.dart
import 'package:dio/dio.dart';

class ApiClient {
  late final Dio _dio;

  ApiClient({
    required String baseUrl,
    required TokenManager tokenManager,
  }) {
    _dio = Dio(
      BaseOptions(
        baseUrl: baseUrl,
        connectTimeout: const Duration(seconds: 30),
        receiveTimeout: const Duration(seconds: 30),
        sendTimeout: const Duration(seconds: 30),
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
        },
      ),
    );

    // Add interceptors
    _dio.interceptors.addAll([
      AuthInterceptor(tokenManager),
      RetryInterceptor(),
      LoggingInterceptor(),  // Only in debug mode
    ]);
  }

  // GET request
  Future<T> get<T>(
    String path, {
    Map<String, dynamic>? queryParameters,
    Options? options,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.get(
        path,
        queryParameters: queryParameters,
        options: options,
      );
      return _handleResponse<T>(response, fromJson);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  // POST request
  Future<T> post<T>(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    Options? options,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.post(
        path,
        data: data,
        queryParameters: queryParameters,
        options: options,
      );
      return _handleResponse<T>(response, fromJson);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  // PUT request
  Future<T> put<T>(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    Options? options,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.put(
        path,
        data: data,
        queryParameters: queryParameters,
        options: options,
      );
      return _handleResponse<T>(response, fromJson);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  // DELETE request
  Future<T> delete<T>(
    String path, {
    dynamic data,
    Map<String, dynamic>? queryParameters,
    Options? options,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final response = await _dio.delete(
        path,
        data: data,
        queryParameters: queryParameters,
        options: options,
      );
      return _handleResponse<T>(response, fromJson);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  // Upload file
  Future<T> uploadFile<T>(
    String path, {
    required String filePath,
    required String fileKey,
    Map<String, dynamic>? data,
    ProgressCallback? onSendProgress,
    T Function(dynamic)? fromJson,
  }) async {
    try {
      final formData = FormData.fromMap({
        ...?data,
        fileKey: await MultipartFile.fromFile(filePath),
      });

      final response = await _dio.post(
        path,
        data: formData,
        onSendProgress: onSendProgress,
      );

      return _handleResponse<T>(response, fromJson);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  // Download file
  Future<void> downloadFile(
    String url,
    String savePath, {
    ProgressCallback? onReceiveProgress,
    CancelToken? cancelToken,
  }) async {
    try {
      await _dio.download(
        url,
        savePath,
        onReceiveProgress: onReceiveProgress,
        cancelToken: cancelToken,
      );
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  T _handleResponse<T>(Response response, T Function(dynamic)? fromJson) {
    if (response.statusCode! >= 200 && response.statusCode! < 300) {
      if (fromJson != null) {
        return fromJson(response.data);
      }
      return response.data as T;
    }
    throw ApiError(
      message: 'Request failed',
      statusCode: response.statusCode,
    );
  }

  ApiError _handleError(DioException error) {
    switch (error.type) {
      case DioExceptionType.connectionTimeout:
      case DioExceptionType.sendTimeout:
      case DioExceptionType.receiveTimeout:
        return TimeoutError();

      case DioExceptionType.connectionError:
        return NetworkError();

      case DioExceptionType.badResponse:
        final statusCode = error.response?.statusCode;
        final message = error.response?.data['message'] ?? 'Unknown error';

        if (statusCode == 401) {
          return UnauthorizedError(message: message);
        } else if (statusCode != null && statusCode >= 500) {
          return ServerError(message: message);
        }

        return ApiError(
          message: message,
          statusCode: statusCode,
          data: error.response?.data,
        );

      default:
        return ApiError(message: error.message ?? 'Unknown error');
    }
  }
}
```

### Step 6: Repository

```dart
// data/repositories/user_repository.dart
class UserRepository {
  final ApiClient _apiClient;

  UserRepository(this._apiClient);

  Future<List<User>> getUsers({int page = 1, int limit = 20}) async {
    return await _apiClient.get<List<User>>(
      ApiEndpoints.users,
      queryParameters: {'page': page, 'limit': limit},
      fromJson: (data) {
        return (data as List).map((json) => User.fromJson(json)).toList();
      },
    );
  }

  Future<User> getUserById(int id) async {
    return await _apiClient.get<User>(
      ApiEndpoints.userById(id),
      fromJson: (data) => User.fromJson(data),
    );
  }

  Future<User> createUser(User user) async {
    return await _apiClient.post<User>(
      ApiEndpoints.users,
      data: user.toJson(),
      fromJson: (data) => User.fromJson(data),
    );
  }

  Future<User> updateUser(int id, User user) async {
    return await _apiClient.put<User>(
      ApiEndpoints.userById(id),
      data: user.toJson(),
      fromJson: (data) => User.fromJson(data),
    );
  }

  Future<void> deleteUser(int id) async {
    await _apiClient.delete(ApiEndpoints.userById(id));
  }
}
```

## Using Retrofit (Type-safe API)

**Android Equivalent:** Retrofit

```yaml
dependencies:
  dio: ^5.4.0
  retrofit: ^4.0.3
  json_annotation: ^4.8.1

dev_dependencies:
  retrofit_generator: ^8.0.6
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
```

```dart
// data/api/api_service.dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';

part 'api_service.g.dart';

@RestApi(baseUrl: 'https://api.example.com/v1')
abstract class ApiService {
  factory ApiService(Dio dio, {String baseUrl}) = _ApiService;

  @GET('/users')
  Future<List<User>> getUsers(
    @Query('page') int page,
    @Query('limit') int limit,
  );

  @GET('/users/{id}')
  Future<User> getUserById(@Path('id') int id);

  @POST('/users')
  Future<User> createUser(@Body() User user);

  @PUT('/users/{id}')
  Future<User> updateUser(
    @Path('id') int id,
    @Body() User user,
  );

  @DELETE('/users/{id}')
  Future<void> deleteUser(@Path('id') int id);

  @POST('/users/{id}/avatar')
  @MultiPart()
  Future<User> uploadAvatar(
    @Path('id') int id,
    @Part(name: 'avatar') File file,
  );

  @GET('/users/search')
  Future<List<User>> searchUsers(
    @Query('q') String query,
    @Queries() Map<String, dynamic> filters,
  );
}

// Generate code
// flutter pub run build_runner build
```

## Comparison with Android

### Android (Retrofit + Coroutines)

```kotlin
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Int): Response<User>

    @POST("users")
    suspend fun createUser(@Body user: User): Response<User>
}

class UserRepository(private val apiService: ApiService) {
    suspend fun getUser(id: Int): Result<User> {
        return try {
            val response = apiService.getUser(id)
            if (response.isSuccessful) {
                Result.success(response.body()!!)
            } else {
                Result.failure(Exception(response.message()))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### Flutter (Dio + Future)

```dart
class ApiService {
  final Dio dio;

  ApiService(this.dio);

  Future<User> getUser(int id) async {
    final response = await dio.get('/users/$id');
    return User.fromJson(response.data);
  }

  Future<User> createUser(User user) async {
    final response = await dio.post('/users', data: user.toJson());
    return User.fromJson(response.data);
  }
}

class UserRepository {
  final ApiService _apiService;

  UserRepository(this._apiService);

  Future<User> getUser(int id) async {
    try {
      return await _apiService.getUser(id);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }
}
```

## Error Handling Patterns

### Pattern 1: Result/Either Pattern

```dart
// Define Result type
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Failure<T> extends Result<T> {
  final String message;
  final Exception? exception;
  const Failure(this.message, {this.exception});
}

// Repository
class UserRepository {
  Future<Result<User>> getUser(int id) async {
    try {
      final user = await _apiClient.get<User>(
        '/users/$id',
        fromJson: (data) => User.fromJson(data),
      );
      return Success(user);
    } on ApiError catch (e) {
      return Failure(e.message, exception: e);
    } catch (e) {
      return Failure('Unknown error', exception: e as Exception);
    }
  }
}

// Usage in ViewModel
Future<void> loadUser(int id) async {
  state = state.copyWith(isLoading: true);

  final result = await _repository.getUser(id);

  switch (result) {
    case Success(:final data):
      state = state.copyWith(user: data, isLoading: false);
    case Failure(:final message):
      state = state.copyWith(error: message, isLoading: false);
  }
}
```

### Pattern 2: Try-Catch in ViewModel

```dart
Future<void> loadUser(int id) async {
  try {
    setState(() => isLoading = true);
    final user = await _repository.getUser(id);
    setState(() {
      this.user = user;
      isLoading = false;
    });
  } on UnauthorizedError {
    // Handle unauthorized
    _navigateToLogin();
  } on NetworkError {
    // Handle network error
    _showError('No internet connection');
  } on ApiError catch (e) {
    // Handle API error
    _showError(e.message);
  } finally {
    setState(() => isLoading = false);
  }
}
```

## Testing the Network Layer

```dart
// Mock API client for testing
class MockApiClient extends Mock implements ApiClient {}

void main() {
  group('UserRepository', () {
    late MockApiClient mockApiClient;
    late UserRepository repository;

    setUp(() {
      mockApiClient = MockApiClient();
      repository = UserRepository(mockApiClient);
    });

    test('getUser returns user on success', () async {
      // Arrange
      final user = User(id: 1, name: 'John', email: 'john@example.com');
      when(mockApiClient.get<User>(any, fromJson: anyNamed('fromJson')))
          .thenAnswer((_) async => user);

      // Act
      final result = await repository.getUserById(1);

      // Assert
      expect(result, user);
      verify(mockApiClient.get<User>(
        ApiEndpoints.userById(1),
        fromJson: anyNamed('fromJson'),
      )).called(1);
    });

    test('getUser throws ApiError on failure', () async {
      // Arrange
      when(mockApiClient.get<User>(any, fromJson: anyNamed('fromJson')))
          .thenThrow(ApiError(message: 'User not found', statusCode: 404));

      // Act & Assert
      expect(
        () => repository.getUserById(1),
        throwsA(isA<ApiError>()),
      );
    });
  });
}
```

## Best Practices

1. **Use a single Dio instance** - Create it once and reuse
2. **Implement proper error handling** - Use custom exceptions
3. **Add interceptors** - For auth, logging, retry
4. **Use models** - Don't work with raw JSON
5. **Repository pattern** - Separate API calls from business logic
6. **Timeout configuration** - Set reasonable timeouts
7. **Cancel requests** - Use CancelToken for long operations
8. **Cache responses** - Use dio_cache_interceptor package
9. **Test thoroughly** - Mock API calls in tests
10. **Environment configuration** - Different URLs for dev/staging/prod

## Next Steps

Now that you understand networking, let's explore state management approaches in [Section 7: State Management](./07-state-management.md).
