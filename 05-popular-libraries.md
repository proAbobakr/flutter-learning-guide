# Section 5: Popular Libraries & Packages

## Introduction

Flutter's package ecosystem is hosted on [pub.dev](https://pub.dev), similar to Maven Central for Android. This section covers the most essential and popular packages you'll need.

## Package Management

### Adding Dependencies

**Android (build.gradle):**
```gradle
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4'
}
```

**Flutter (pubspec.yaml):**
```yaml
dependencies:
  dio: ^5.4.0
  provider: ^6.1.1

dev_dependencies:
  build_runner: ^2.4.7
```

Then run:
```bash
flutter pub get  # Similar to Gradle sync
```

## Essential Packages by Category

### 1. State Management

| Package | Android Equivalent | Use Case |
|---------|-------------------|----------|
| provider | ViewModel + LiveData | Simple, recommended by Flutter team |
| riverpod | Hilt + ViewModel | Provider with better syntax |
| bloc | ViewModel + LiveData | Complex state, predictable patterns |
| get | - | Simple, includes navigation |
| mobx | MobX Android | Reactive state management |

#### Provider (Recommended for beginners)

```yaml
dependencies:
  provider: ^6.1.1
```

```dart
// 1. Create a ChangeNotifier (similar to ViewModel)
class CounterProvider extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();  // Similar to LiveData.postValue()
  }
}

// 2. Provide it
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterProvider(),
      child: MyApp(),
    ),
  );
}

// 3. Consume it
class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = Provider.of<CounterProvider>(context);
    // Or: final counter = context.watch<CounterProvider>();

    return Column(
      children: [
        Text('Count: ${counter.count}'),
        ElevatedButton(
          onPressed: counter.increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

#### Riverpod (Modern alternative)

```yaml
dependencies:
  flutter_riverpod: ^2.4.9
```

```dart
// 1. Define provider
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() => state++;
}

// 2. Wrap app
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

// 3. Consume
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

### 2. Networking

| Package | Android Equivalent | Description |
|---------|-------------------|-------------|
| dio | Retrofit + OkHttp | HTTP client with interceptors |
| http | HttpURLConnection | Simple HTTP requests |
| retrofit | Retrofit | Type-safe HTTP client (code generation) |

#### Dio (Most popular)

```yaml
dependencies:
  dio: ^5.4.0
```

```dart
class ApiService {
  final Dio _dio = Dio(
    BaseOptions(
      baseUrl: 'https://api.example.com',
      connectTimeout: Duration(seconds: 5),
      receiveTimeout: Duration(seconds: 3),
    ),
  );

  ApiService() {
    // Add interceptors
    _dio.interceptors.add(LogInterceptor(
      requestBody: true,
      responseBody: true,
    ));

    _dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) {
        // Add auth token
        options.headers['Authorization'] = 'Bearer $token';
        return handler.next(options);
      },
      onError: (error, handler) {
        // Handle errors
        return handler.next(error);
      },
    ));
  }

  Future<User> getUser(int id) async {
    try {
      final response = await _dio.get('/users/$id');
      return User.fromJson(response.data);
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }

  Future<List<User>> getUsers() async {
    final response = await _dio.get('/users');
    return (response.data as List)
        .map((json) => User.fromJson(json))
        .toList();
  }

  Future<User> createUser(User user) async {
    final response = await _dio.post('/users', data: user.toJson());
    return User.fromJson(response.data);
  }
}
```

### 3. JSON Serialization

| Package | Android Equivalent | Description |
|---------|-------------------|-------------|
| json_serializable | Gson / Moshi | Code generation for JSON |
| freezed | AutoValue | Immutable data classes |
| built_value | - | Immutable value types |

#### json_serializable

```yaml
dependencies:
  json_annotation: ^4.8.1

dev_dependencies:
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
```

```dart
import 'package:json_annotation/json_annotation.dart';

part 'user.g.dart';

@JsonSerializable()
class User {
  final int id;
  final String name;
  @JsonKey(name: 'email_address')  // Custom JSON key
  final String email;

  User({required this.id, required this.name, required this.email});

  // Code generated methods
  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

Run code generation:
```bash
flutter pub run build_runner build
# Or watch for changes
flutter pub run build_runner watch
```

#### Freezed (Recommended for data classes)

```yaml
dependencies:
  freezed_annotation: ^2.4.1

dev_dependencies:
  build_runner: ^2.4.7
  freezed: ^2.4.6
  json_serializable: ^6.7.1
```

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  const factory User({
    required int id,
    required String name,
    String? email,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}

// Provides: copyWith, ==, hashCode, toString, fromJson, toJson
```

### 4. Local Storage

| Package | Android Equivalent | Use Case |
|---------|-------------------|----------|
| shared_preferences | SharedPreferences | Simple key-value storage |
| hive | Room | NoSQL database |
| sqflite | Room | SQLite database |
| isar | Realm | High-performance database |

#### shared_preferences

```yaml
dependencies:
  shared_preferences: ^2.2.2
```

```dart
class PreferencesService {
  static const String _keyToken = 'auth_token';
  static const String _keyUserId = 'user_id';

  Future<void> saveToken(String token) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_keyToken, token);
  }

  Future<String?> getToken() async {
    final prefs = await SharedPreferences.getInstance();
    return prefs.getString(_keyToken);
  }

  Future<void> clearAll() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.clear();
  }
}
```

#### Hive (Fast NoSQL)

```yaml
dependencies:
  hive: ^2.2.3
  hive_flutter: ^1.1.0

dev_dependencies:
  hive_generator: ^2.0.1
  build_runner: ^2.4.7
```

```dart
// 1. Define model
import 'package:hive/hive.dart';

part 'user.g.dart';

@HiveType(typeId: 0)
class User extends HiveObject {
  @HiveField(0)
  final int id;

  @HiveField(1)
  final String name;

  @HiveField(2)
  final String? email;

  User({required this.id, required this.name, this.email});
}

// 2. Initialize
void main() async {
  await Hive.initFlutter();
  Hive.registerAdapter(UserAdapter());
  await Hive.openBox<User>('users');
  runApp(MyApp());
}

// 3. Use
class UserRepository {
  final Box<User> _box = Hive.box<User>('users');

  Future<void> addUser(User user) async {
    await _box.put(user.id, user);
  }

  User? getUser(int id) {
    return _box.get(id);
  }

  List<User> getAllUsers() {
    return _box.values.toList();
  }

  Future<void> deleteUser(int id) async {
    await _box.delete(id);
  }
}
```

### 5. Dependency Injection

| Package | Android Equivalent | Description |
|---------|-------------------|-------------|
| get_it | Hilt/Dagger | Service locator |
| injectable | Hilt | Code generation for get_it |
| provider | Hilt (simpler) | DI via widget tree |

#### get_it + injectable

```yaml
dependencies:
  get_it: ^7.6.4
  injectable: ^2.3.2

dev_dependencies:
  injectable_generator: ^2.4.1
  build_runner: ^2.4.7
```

```dart
// 1. Setup (injection.dart)
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';

final getIt = GetIt.instance;

@InjectableInit()
void configureDependencies() => getIt.init();

// 2. Define dependencies
@singleton
class ApiService {
  final Dio dio;
  ApiService(this.dio);
}

@injectable
class UserRepository {
  final ApiService _apiService;
  UserRepository(this._apiService);

  Future<User> getUser(int id) => _apiService.getUser(id);
}

@injectable
class UserViewModel {
  final UserRepository _repository;
  UserViewModel(this._repository);

  Future<void> loadUser(int id) async {
    final user = await _repository.getUser(id);
    // Update state
  }
}

// 3. Initialize in main
void main() {
  configureDependencies();
  runApp(MyApp());
}

// 4. Use
class UserScreen extends StatelessWidget {
  final viewModel = getIt<UserViewModel>();

  @override
  Widget build(BuildContext context) {
    // Use viewModel
  }
}
```

### 6. Navigation

| Package | Android Equivalent | Description |
|---------|-------------------|-------------|
| go_router | Navigation Component | Declarative routing |
| auto_route | Navigation Component | Code generation routing |

#### go_router

```yaml
dependencies:
  go_router: ^13.0.0
```

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
    GoRoute(
      path: '/user/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return UserScreen(userId: int.parse(id));
      },
    ),
    GoRoute(
      path: '/settings',
      builder: (context, state) => SettingsScreen(),
    ),
  ],
);

void main() {
  runApp(MaterialApp.router(
    routerConfig: router,
  ));
}

// Navigate
context.go('/user/123');
context.push('/settings');
context.pop();
```

### 7. Image Loading & Caching

| Package | Android Equivalent | Description |
|---------|-------------------|-------------|
| cached_network_image | Glide / Picasso | Image caching |
| flutter_cache_manager | - | General file caching |

```yaml
dependencies:
  cached_network_image: ^3.3.1
```

```dart
CachedNetworkImage(
  imageUrl: 'https://example.com/image.jpg',
  placeholder: (context, url) => CircularProgressIndicator(),
  errorWidget: (context, url, error) => Icon(Icons.error),
  fit: BoxFit.cover,
)
```

### 8. Database (SQL)

```yaml
dependencies:
  sqflite: ^2.3.0
  path: ^1.8.3
```

```dart
class DatabaseService {
  static Database? _database;

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  Future<Database> _initDatabase() async {
    final dbPath = await getDatabasesPath();
    final path = join(dbPath, 'app_database.db');

    return await openDatabase(
      path,
      version: 1,
      onCreate: (db, version) async {
        await db.execute('''
          CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            email TEXT
          )
        ''');
      },
    );
  }

  Future<int> insertUser(User user) async {
    final db = await database;
    return await db.insert('users', user.toMap());
  }

  Future<List<User>> getUsers() async {
    final db = await database;
    final maps = await db.query('users');
    return maps.map((map) => User.fromMap(map)).toList();
  }

  Future<int> updateUser(User user) async {
    final db = await database;
    return await db.update(
      'users',
      user.toMap(),
      where: 'id = ?',
      whereArgs: [user.id],
    );
  }

  Future<int> deleteUser(int id) async {
    final db = await database;
    return await db.delete(
      'users',
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}
```

### 9. Testing

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.4
  bloc_test: ^9.1.5
  integration_test:
    sdk: flutter
```

```dart
// Unit test
void main() {
  group('Counter', () {
    test('starts at 0', () {
      final counter = Counter();
      expect(counter.value, 0);
    });

    test('increments', () {
      final counter = Counter();
      counter.increment();
      expect(counter.value, 1);
    });
  });
}

// Widget test
void main() {
  testWidgets('Counter increments smoke test', (WidgetTester tester) async {
    await tester.pumpWidget(MyApp());

    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);

    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });
}
```

### 10. Utilities

#### intl (Internationalization)

```yaml
dependencies:
  intl: ^0.18.1
```

```dart
// Date formatting
import 'package:intl/intl.dart';

final formatter = DateFormat('yyyy-MM-dd');
final dateString = formatter.format(DateTime.now());

// Number formatting
final currencyFormatter = NumberFormat.currency(symbol: '\$');
final price = currencyFormatter.format(99.99);  // $99.99
```

#### logger

```yaml
dependencies:
  logger: ^2.0.2
```

```dart
final logger = Logger(
  printer: PrettyPrinter(),
);

logger.d('Debug message');
logger.i('Info message');
logger.w('Warning message');
logger.e('Error message');
```

#### equatable (Easy equality)

```yaml
dependencies:
  equatable: ^2.0.5
```

```dart
class User extends Equatable {
  final int id;
  final String name;

  const User(this.id, this.name);

  @override
  List<Object> get props => [id, name];
}

// Now you get == and hashCode for free
final user1 = User(1, 'John');
final user2 = User(1, 'John');
print(user1 == user2);  // true
```

### 11. Async & Reactive

#### rxdart (RxJava equivalent)

```yaml
dependencies:
  rxdart: ^0.27.7
```

```dart
// BehaviorSubject (similar to LiveData)
final _counterSubject = BehaviorSubject<int>.seeded(0);

Stream<int> get counterStream => _counterSubject.stream;
int get currentValue => _counterSubject.value;

void increment() {
  _counterSubject.add(_counterSubject.value + 1);
}

void dispose() {
  _counterSubject.close();
}

// Operators
stream
  .debounceTime(Duration(milliseconds: 500))
  .distinctUntilChanged()
  .listen((value) {
    print(value);
  });
```

### 12. Platform Integration

```yaml
dependencies:
  url_launcher: ^6.2.2  # Open URLs, make calls
  image_picker: ^1.0.5  # Pick images from gallery/camera
  path_provider: ^2.1.1  # Get app directories
  permission_handler: ^11.1.0  # Handle permissions
  device_info_plus: ^9.1.1  # Device information
  package_info_plus: ^5.0.1  # App information
```

### 13. UI Enhancement

```yaml
dependencies:
  shimmer: ^3.0.0  # Loading skeleton
  lottie: ^2.7.0  # Lottie animations
  flutter_svg: ^2.0.9  # SVG support
  google_fonts: ^6.1.0  # Google Fonts
```

```dart
// Shimmer
Shimmer.fromColors(
  baseColor: Colors.grey[300]!,
  highlightColor: Colors.grey[100]!,
  child: Container(width: 200, height: 100),
)

// Lottie
Lottie.asset('assets/animations/loading.json')

// Google Fonts
Text(
  'Hello',
  style: GoogleFonts.roboto(fontSize: 24),
)
```

## Recommended Package Combinations

### Minimal App (Simple)
```yaml
dependencies:
  flutter:
    sdk: flutter
  provider: ^6.1.1
  http: ^1.1.2
  shared_preferences: ^2.2.2
```

### Standard App (Recommended)
```yaml
dependencies:
  # State management
  provider: ^6.1.1

  # Networking
  dio: ^5.4.0

  # Serialization
  json_annotation: ^4.8.1
  freezed_annotation: ^2.4.1

  # Storage
  shared_preferences: ^2.2.2
  hive: ^2.2.3

  # Navigation
  go_router: ^13.0.0

  # DI
  get_it: ^7.6.4

  # Utilities
  equatable: ^2.0.5
  intl: ^0.18.1

dev_dependencies:
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
  freezed: ^2.4.6
```

### Enterprise App (Full-featured)
```yaml
dependencies:
  # State management
  riverpod: ^2.4.9

  # Networking
  dio: ^5.4.0
  retrofit: ^4.0.3

  # Serialization
  freezed_annotation: ^2.4.1

  # Database
  isar: ^3.1.0

  # DI
  injectable: ^2.3.2

  # Navigation
  auto_route: ^7.8.4

  # Testing
  mockito: ^5.4.4
```

## Package Discovery

- **Official packages**: [pub.dev/publishers/flutter.dev](https://pub.dev/publishers/flutter.dev)
- **Most popular**: [pub.dev/packages?sort=popularity](https://pub.dev/packages?sort=popularity)
- **Search**: [pub.dev](https://pub.dev)
- **Awesome Flutter**: [GitHub - Awesome Flutter](https://github.com/Solido/awesome-flutter)

## Next Steps

Now that you know the essential packages, let's dive deep into networking in [Section 6: Network Layer](./06-network-layer.md).
