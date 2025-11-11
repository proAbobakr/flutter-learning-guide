# Section 9: Best Practices & Tips

## Introduction

This section covers essential best practices, performance optimization, testing strategies, and common pitfalls to help you write better Flutter code.

## Performance Optimization

### 1. Use const Constructors

**❌ Bad:**
```dart
Widget build(BuildContext context) {
  return Container(
    child: Text('Hello'),
  );
}
```

**✅ Good:**
```dart
Widget build(BuildContext context) {
  return const Container(
    child: Text('Hello'),
  );
}
```

**Why:** `const` widgets are created at compile-time and reused, reducing rebuilds and memory usage.

### 2. Extract Widgets

**❌ Bad:**
```dart
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Container(
          padding: EdgeInsets.all(16),
          child: Row(
            children: [
              Icon(Icons.person),
              Text('User'),
            ],
          ),
        ),
        // More complex widgets...
      ],
    );
  }
}
```

**✅ Good:**
```dart
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        UserHeader(),
        // More widgets...
      ],
    );
  }
}

class UserHeader extends StatelessWidget {
  const UserHeader({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: const [
          Icon(Icons.person),
          Text('User'),
        ],
      ),
    );
  }
}
```

**Why:** Smaller widgets = smaller rebuild scope = better performance

### 3. Use ListView.builder for Long Lists

**❌ Bad:**
```dart
ListView(
  children: items.map((item) => ListTile(title: Text(item))).toList(),
)
```

**✅ Good:**
```dart
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
)
```

**Why:** Builder only creates visible items, lazy loading for better performance

### 4. Avoid Rebuilding the Entire Tree

**❌ Bad:**
```dart
class HomeScreen extends StatefulWidget {
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Home'),  // Rebuilds unnecessarily
      ),
      body: Column(
        children: [
          ExpensiveWidget(),  // Rebuilds unnecessarily
          Text('Counter: $_counter'),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _counter++),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

**✅ Good:**
```dart
class HomeScreen extends StatefulWidget {
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home'),  // const = won't rebuild
      ),
      body: Column(
        children: [
          const ExpensiveWidget(),  // const = won't rebuild
          CounterDisplay(counter: _counter),  // Only this rebuilds
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _counter++),
        child: const Icon(Icons.add),
      ),
    );
  }
}

class CounterDisplay extends StatelessWidget {
  final int counter;

  const CounterDisplay({Key? key, required this.counter}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text('Counter: $counter');
  }
}
```

### 5. Use RepaintBoundary for Expensive Widgets

```dart
RepaintBoundary(
  child: ExpensiveAnimationWidget(),
)
```

**Why:** Isolates repaints to prevent affecting other widgets

### 6. Optimize Images

```dart
// ❌ Bad: Large images
Image.asset('assets/large_image.jpg')

// ✅ Good: Properly sized images
Image.asset(
  'assets/optimized_image.jpg',
  cacheWidth: 200,  // Decode to specific size
  cacheHeight: 200,
)

// Use CachedNetworkImage for network images
CachedNetworkImage(
  imageUrl: url,
  placeholder: (context, url) => CircularProgressIndicator(),
  errorWidget: (context, url, error) => Icon(Icons.error),
)
```

### 7. Avoid Expensive Operations in build()

**❌ Bad:**
```dart
@override
Widget build(BuildContext context) {
  final processedData = expensiveComputation(data);  // Runs every rebuild!
  return Text(processedData);
}
```

**✅ Good:**
```dart
late String _processedData;

@override
void initState() {
  super.initState();
  _processedData = expensiveComputation(data);  // Once
}

@override
Widget build(BuildContext context) {
  return Text(_processedData);
}
```

### 8. Use Keys Appropriately

```dart
// Use keys when order/identity matters
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(
      key: ValueKey(items[index].id),  // Stable identity
      title: Text(items[index].name),
    );
  },
)
```

---

## Code Organization

### 1. Feature-First Structure

```
lib/
├── features/
│   ├── authentication/
│   │   ├── data/
│   │   │   ├── models/
│   │   │   ├── repositories/
│   │   │   └── data_sources/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   └── use_cases/
│   │   └── presentation/
│   │       ├── screens/
│   │       ├── widgets/
│   │       └── providers/
│   ├── home/
│   └── profile/
├── core/
│   ├── constants/
│   ├── theme/
│   ├── utils/
│   └── widgets/
└── main.dart
```

### 2. Naming Conventions

```dart
// Files: snake_case
user_profile_screen.dart
user_repository.dart

// Classes: PascalCase
class UserProfileScreen {}
class UserRepository {}

// Variables/Functions: camelCase
String userName = 'John';
void fetchUserData() {}

// Constants: lowerCamelCase or UPPER_SNAKE_CASE
const double defaultPadding = 16.0;
const int MAX_RETRY_COUNT = 3;

// Private: prefix with _
String _privateVariable;
void _privateMethod() {}
```

### 3. Separate Business Logic from UI

**❌ Bad:**
```dart
class UserScreen extends StatefulWidget {
  @override
  State<UserScreen> createState() => _UserScreenState();
}

class _UserScreenState extends State<UserScreen> {
  User? user;

  @override
  void initState() {
    super.initState();
    _loadUser();
  }

  Future<void> _loadUser() async {
    // API call directly in widget
    final response = await http.get(Uri.parse('https://api.example.com/user'));
    setState(() {
      user = User.fromJson(jsonDecode(response.body));
    });
  }

  @override
  Widget build(BuildContext context) {
    return Text(user?.name ?? 'Loading...');
  }
}
```

**✅ Good:**
```dart
// Repository
class UserRepository {
  Future<User> getUser(int id) async {
    final response = await http.get(Uri.parse('https://api.example.com/user/$id'));
    return User.fromJson(jsonDecode(response.body));
  }
}

// Provider/ViewModel
class UserProvider extends ChangeNotifier {
  final UserRepository _repository;
  User? _user;
  bool _isLoading = false;

  User? get user => _user;
  bool get isLoading => _isLoading;

  UserProvider(this._repository);

  Future<void> loadUser(int id) async {
    _isLoading = true;
    notifyListeners();

    try {
      _user = await _repository.getUser(id);
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}

// UI
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userProvider = context.watch<UserProvider>();

    if (userProvider.isLoading) {
      return CircularProgressIndicator();
    }

    return Text(userProvider.user?.name ?? 'No user');
  }
}
```

---

## Testing

### 1. Unit Tests

```dart
// test/repositories/user_repository_test.dart
void main() {
  group('UserRepository', () {
    late UserRepository repository;
    late MockApiClient mockApiClient;

    setUp(() {
      mockApiClient = MockApiClient();
      repository = UserRepository(mockApiClient);
    });

    test('getUser returns user on success', () async {
      // Arrange
      final user = User(id: 1, name: 'John');
      when(mockApiClient.get(any)).thenAnswer((_) async => user);

      // Act
      final result = await repository.getUser(1);

      // Assert
      expect(result, user);
      verify(mockApiClient.get('/users/1')).called(1);
    });

    test('getUser throws exception on failure', () async {
      // Arrange
      when(mockApiClient.get(any)).thenThrow(Exception('Network error'));

      // Act & Assert
      expect(
        () => repository.getUser(1),
        throwsException,
      );
    });
  });
}
```

### 2. Widget Tests

```dart
// test/widgets/user_card_test.dart
void main() {
  testWidgets('UserCard displays user information', (tester) async {
    // Arrange
    final user = User(id: 1, name: 'John', email: 'john@example.com');

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: UserCard(user: user),
      ),
    );

    // Assert
    expect(find.text('John'), findsOneWidget);
    expect(find.text('john@example.com'), findsOneWidget);
  });

  testWidgets('UserCard calls onTap when tapped', (tester) async {
    // Arrange
    bool tapped = false;
    final user = User(id: 1, name: 'John');

    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: UserCard(
          user: user,
          onTap: () => tapped = true,
        ),
      ),
    );

    await tester.tap(find.byType(UserCard));
    await tester.pump();

    // Assert
    expect(tapped, true);
  });
}
```

### 3. Integration Tests

```dart
// integration_test/app_test.dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('Complete user flow', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    // Tap login button
    await tester.tap(find.text('Login'));
    await tester.pumpAndSettle();

    // Enter credentials
    await tester.enterText(find.byType(TextField).first, 'user@example.com');
    await tester.enterText(find.byType(TextField).last, 'password');
    await tester.tap(find.text('Submit'));
    await tester.pumpAndSettle();

    // Verify home screen
    expect(find.text('Welcome'), findsOneWidget);
  });
}
```

### 4. Golden Tests (Screenshot Testing)

```dart
void main() {
  testWidgets('UserCard golden test', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: UserCard(user: User(id: 1, name: 'John')),
      ),
    );

    await expectLater(
      find.byType(UserCard),
      matchesGoldenFile('goldens/user_card.png'),
    );
  });
}
```

---

## Error Handling

### 1. Try-Catch with Specific Exceptions

```dart
Future<User> loadUser(int id) async {
  try {
    return await _repository.getUser(id);
  } on NetworkException catch (e) {
    _showError('No internet connection');
    rethrow;
  } on UnauthorizedException catch (e) {
    _navigateToLogin();
    rethrow;
  } on NotFoundException catch (e) {
    _showError('User not found');
    rethrow;
  } catch (e) {
    _showError('An unexpected error occurred');
    rethrow;
  }
}
```

### 2. Global Error Handler

```dart
void main() {
  FlutterError.onError = (details) {
    // Log to crash reporting service
    FirebaseCrashlytics.instance.recordFlutterError(details);
  };

  runZonedGuarded(() {
    runApp(MyApp());
  }, (error, stack) {
    // Handle async errors
    FirebaseCrashlytics.instance.recordError(error, stack);
  });
}
```

---

## Common Pitfalls

### 1. Not Disposing Resources

**❌ Bad:**
```dart
class MyScreen extends StatefulWidget {
  @override
  State<MyScreen> createState() => _MyScreenState();
}

class _MyScreenState extends State<MyScreen> {
  late TextEditingController _controller;
  late AnimationController _animController;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
    _animController = AnimationController(vsync: this);
  }

  // Missing dispose() - Memory leak!
}
```

**✅ Good:**
```dart
class _MyScreenState extends State<MyScreen> {
  late TextEditingController _controller;
  late AnimationController _animController;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController();
    _animController = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    _animController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### 2. Using BuildContext After Async Gap

**❌ Bad:**
```dart
Future<void> saveData() async {
  await repository.save(data);
  Navigator.pop(context);  // Context might be invalid!
}
```

**✅ Good:**
```dart
Future<void> saveData() async {
  await repository.save(data);
  if (mounted) {  // Check if still mounted
    Navigator.pop(context);
  }
}
```

### 3. Not Using Keys for Stateful Widgets in Lists

**❌ Bad:**
```dart
ListView(
  children: items.map((item) => StatefulItemWidget(item)).toList(),
)
```

**✅ Good:**
```dart
ListView(
  children: items.map((item) {
    return StatefulItemWidget(
      key: ValueKey(item.id),
      item: item,
    );
  }).toList(),
)
```

### 4. Calling setState During Build

**❌ Bad:**
```dart
@override
Widget build(BuildContext context) {
  if (someCondition) {
    setState(() {});  // Error!
  }
  return Container();
}
```

**✅ Good:**
```dart
@override
Widget build(BuildContext context) {
  if (someCondition) {
    WidgetsBinding.instance.addPostFrameCallback((_) {
      setState(() {});
    });
  }
  return Container();
}
```

### 5. Forgetting to Call super in Lifecycle Methods

**❌ Bad:**
```dart
@override
void initState() {
  // Missing super.initState()!
  loadData();
}
```

**✅ Good:**
```dart
@override
void initState() {
  super.initState();  // Always first
  loadData();
}

@override
void dispose() {
  cleanup();
  super.dispose();  // Always last
}
```

---

## Security Best Practices

### 1. Never Store Sensitive Data in Plain Text

```dart
// ❌ Bad
SharedPreferences.setString('password', password);

// ✅ Good: Use flutter_secure_storage
final storage = FlutterSecureStorage();
await storage.write(key: 'auth_token', value: token);
```

### 2. Validate User Input

```dart
String? validateEmail(String? value) {
  if (value == null || value.isEmpty) {
    return 'Email is required';
  }
  final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
  if (!emailRegex.hasMatch(value)) {
    return 'Invalid email format';
  }
  return null;
}

TextField(
  validator: validateEmail,
)
```

### 3. Use HTTPS Only

```dart
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',  // Always HTTPS
));
```

---

## Accessibility

```dart
// Add semantic labels
Semantics(
  label: 'Profile picture',
  child: Image.asset('assets/avatar.jpg'),
)

// Use semantic buttons
IconButton(
  icon: Icon(Icons.favorite),
  tooltip: 'Add to favorites',  // For accessibility
  onPressed: () {},
)

// Proper text scaling
Text(
  'Hello',
  style: Theme.of(context).textTheme.headline6,  // Respects user font size
)
```

---

## Debugging Tips

### 1. Use Debugger and Breakpoints

```dart
debugger();  // Pauses execution
```

### 2. Print Debugging

```dart
print('User: $user');
debugPrint('This is debug print');  // Better for large outputs
```

### 3. Flutter Inspector

- Widget tree visualization
- Layout explorer
- Performance overlay

### 4. Performance Profiling

```bash
flutter run --profile
```

Then use DevTools for detailed profiling.

---

## Useful Tools & Extensions

### VS Code Extensions
- Flutter
- Dart
- Awesome Flutter Snippets
- Error Lens
- GitLens

### Android Studio Plugins
- Flutter
- Dart
- Rainbow Brackets
- Key Promoter X

### Command Line Tools
```bash
# Analyze code
flutter analyze

# Format code
flutter format .

# Check dependencies
flutter pub outdated

# Clean build
flutter clean

# Generate icons
flutter pub run flutter_launcher_icons:main

# Generate splash screen
flutter pub run flutter_native_splash:create
```

---

## Code Quality

### 1. Enable Strict Linting

**analysis_options.yaml:**
```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    prefer_const_constructors: true
    prefer_final_fields: true
    avoid_print: true
    prefer_single_quotes: true
    sort_pub_dependencies: true
```

### 2. Use Code Generation

```yaml
dev_dependencies:
  build_runner: ^2.4.7
  json_serializable: ^6.7.1
  freezed: ^2.4.6
```

### 3. Write Documentation

```dart
/// Fetches user data from the API.
///
/// Throws [NetworkException] if network is unavailable.
/// Throws [UnauthorizedException] if token is invalid.
///
/// Returns [User] object on success.
Future<User> fetchUser(int userId) async {
  // Implementation
}
```

---

## Performance Checklist

- [ ] Use const constructors where possible
- [ ] Extract widgets to reduce rebuild scope
- [ ] Use ListView.builder for long lists
- [ ] Optimize images (compress, cache)
- [ ] Avoid expensive operations in build()
- [ ] Use keys for stateful widgets in lists
- [ ] Profile your app regularly
- [ ] Minimize widget tree depth
- [ ] Use RepaintBoundary for complex animations
- [ ] Lazy load data

## Testing Checklist

- [ ] Unit tests for business logic
- [ ] Widget tests for UI components
- [ ] Integration tests for critical flows
- [ ] Test error scenarios
- [ ] Test async operations
- [ ] Mock external dependencies
- [ ] Aim for >80% code coverage

## Before Release Checklist

- [ ] Run flutter analyze
- [ ] Run all tests
- [ ] Test on real devices
- [ ] Test different screen sizes
- [ ] Test slow network conditions
- [ ] Handle all error cases
- [ ] Add analytics
- [ ] Add crash reporting
- [ ] Optimize app size
- [ ] Test deep linking
- [ ] Update app icons and splash screen
- [ ] Review permissions

---

## Congratulations!

You've completed the Flutter learning guide for Kotlin Android developers. You now have a solid foundation to build production-ready Flutter apps.

### Next Steps

1. **Build a real app** - Apply what you've learned
2. **Contribute to open source** - Flutter packages on pub.dev
3. **Join the community** - Flutter Discord, Reddit, Stack Overflow
4. **Stay updated** - Follow Flutter blog and releases
5. **Explore advanced topics** - Custom painting, animations, platform channels

### Resources

- [Flutter Documentation](https://flutter.dev/docs)
- [Flutter Samples](https://github.com/flutter/samples)
- [Flutter Codelabs](https://flutter.dev/docs/codelabs)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)
- [pub.dev Packages](https://pub.dev)

**Happy Flutter Development! 🚀**
