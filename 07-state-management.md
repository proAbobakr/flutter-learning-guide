# Section 7: State Management

## Introduction

State management is one of the most important topics in Flutter development. Coming from Android, you're familiar with concepts like ViewModel, LiveData, and StateFlow. Flutter offers multiple approaches, each with its own philosophy and use cases.

## Android State Management Review

**Android (MVVM with ViewModel):**
```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user

    private val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading

    fun loadUser(id: Int) {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                val user = repository.getUser(id)
                _user.value = user
            } catch (e: Exception) {
                // Handle error
            } finally {
                _isLoading.value = false
            }
        }
    }
}
```

## Flutter State Management Options

### 1. setState (Basic - Built-in)

**Use Case:** Simple, local widget state

**Android Equivalent:** View state (not ViewModel)

```dart
class CounterScreen extends StatefulWidget {
  @override
  State<CounterScreen> createState() => _CounterScreenState();
}

class _CounterScreenState extends State<CounterScreen> {
  int _counter = 0;
  bool _isLoading = false;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  Future<void> _loadData() async {
    setState(() {
      _isLoading = true;
    });

    try {
      // Fetch data
      await Future.delayed(Duration(seconds: 2));
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: _isLoading
            ? CircularProgressIndicator()
            : Text('Count: $_counter'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

**Pros:**
- ✅ Simple and straightforward
- ✅ No external dependencies
- ✅ Good for simple, local state

**Cons:**
- ❌ State tied to widget
- ❌ Hard to share state
- ❌ No separation of concerns
- ❌ Hard to test

**When to use:** Simple counters, toggles, form fields within a single widget

---

## 2. Provider (Recommended by Flutter Team)

**Android Equivalent:** ViewModel + LiveData

```yaml
dependencies:
  provider: ^6.1.1
```

### Basic Example

```dart
// 1. Create ChangeNotifier (like ViewModel)
class CounterProvider extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();  // Like LiveData.postValue()
  }

  void decrement() {
    _count--;
    notifyListeners();
  }

  void reset() {
    _count = 0;
    notifyListeners();
  }
}

// 2. Provide it at app level
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterProvider(),
      child: MyApp(),
    ),
  );
}

// 3. Consume in widgets
class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Watch for changes (rebuilds on change)
    final counter = context.watch<CounterProvider>();

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Text('Count: ${counter.count}'),
      ),
      floatingActionButton: FloatingActionButton(
        // Read without rebuilding
        onPressed: () => context.read<CounterProvider>().increment(),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### Advanced Example (Real-world)

```dart
// User Provider
class UserProvider extends ChangeNotifier {
  final UserRepository _repository;

  UserProvider(this._repository);

  User? _user;
  bool _isLoading = false;
  String? _error;

  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadUser(int id) async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _user = await _repository.getUser(id);
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  void logout() {
    _user = null;
    notifyListeners();
  }
}

// Multiple providers
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => CounterProvider()),
        ChangeNotifierProvider(
          create: (_) => UserProvider(UserRepository()),
        ),
        ChangeNotifierProvider(create: (_) => ThemeProvider()),
      ],
      child: MyApp(),
    ),
  );
}

// Consuming
class UserProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final userProvider = context.watch<UserProvider>();

    if (userProvider.isLoading) {
      return Center(child: CircularProgressIndicator());
    }

    if (userProvider.error != null) {
      return Center(child: Text('Error: ${userProvider.error}'));
    }

    final user = userProvider.user;
    if (user == null) {
      return Center(child: Text('No user'));
    }

    return Column(
      children: [
        Text('Name: ${user.name}'),
        Text('Email: ${user.email}'),
        ElevatedButton(
          onPressed: () => context.read<UserProvider>().logout(),
          child: Text('Logout'),
        ),
      ],
    );
  }
}
```

**Pros:**
- ✅ Recommended by Flutter team
- ✅ Simple API
- ✅ Good for small to medium apps
- ✅ Easy to learn
- ✅ Good ecosystem support

**Cons:**
- ❌ Boilerplate for complex states
- ❌ No built-in async state handling
- ❌ Can be verbose

**When to use:** Most apps, especially if you're new to Flutter

---

## 3. Riverpod (Modern Provider)

**Android Equivalent:** Hilt + ViewModel (more powerful)

```yaml
dependencies:
  flutter_riverpod: ^2.4.9
```

### Basic Example

```dart
// 1. Define provider (no BuildContext needed!)
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}

// 2. Wrap app with ProviderScope
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}

// 3. Consume with ConsumerWidget
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Text('Count: $count'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).increment(),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### Advanced Example

```dart
// User state
@freezed
class UserState with _$UserState {
  const factory UserState.initial() = _Initial;
  const factory UserState.loading() = _Loading;
  const factory UserState.loaded(User user) = _Loaded;
  const factory UserState.error(String message) = _Error;
}

// User provider
final userProvider = StateNotifierProvider.autoDispose
    .family<UserNotifier, UserState, int>((ref, userId) {
  return UserNotifier(ref.read(userRepositoryProvider), userId);
});

class UserNotifier extends StateNotifier<UserState> {
  final UserRepository _repository;
  final int _userId;

  UserNotifier(this._repository, this._userId)
      : super(const UserState.initial()) {
    loadUser();
  }

  Future<void> loadUser() async {
    state = const UserState.loading();
    try {
      final user = await _repository.getUser(_userId);
      state = UserState.loaded(user);
    } catch (e) {
      state = UserState.error(e.toString());
    }
  }
}

// Usage
class UserProfileScreen extends ConsumerWidget {
  final int userId;

  const UserProfileScreen({required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userState = ref.watch(userProvider(userId));

    return userState.when(
      initial: () => SizedBox(),
      loading: () => Center(child: CircularProgressIndicator()),
      loaded: (user) => Column(
        children: [
          Text('Name: ${user.name}'),
          Text('Email: ${user.email}'),
        ],
      ),
      error: (message) => Center(child: Text('Error: $message')),
    );
  }
}

// Combining providers
final userPostsProvider = FutureProvider.autoDispose
    .family<List<Post>, int>((ref, userId) async {
  // Automatically refetches if user changes
  final user = ref.watch(userProvider(userId));

  return user.maybeWhen(
    loaded: (user) => ref.read(postRepositoryProvider).getUserPosts(user.id),
    orElse: () => [],
  );
});
```

**Pros:**
- ✅ Compile-time safety
- ✅ No BuildContext needed
- ✅ Better testing
- ✅ Automatic disposal
- ✅ Provider dependencies
- ✅ Family and autoDispose modifiers

**Cons:**
- ❌ Steeper learning curve than Provider
- ❌ More concepts to learn

**When to use:** Medium to large apps, when you want type safety and testability

---

## 4. BLoC (Business Logic Component)

**Android Equivalent:** ViewModel + StateFlow/LiveData (more structured)

```yaml
dependencies:
  flutter_bloc: ^8.1.3
```

### Basic Example

```dart
// 1. Define events
abstract class CounterEvent {}

class IncrementEvent extends CounterEvent {}
class DecrementEvent extends CounterEvent {}
class ResetEvent extends CounterEvent {}

// 2. Define states
abstract class CounterState {
  final int count;
  CounterState(this.count);
}

class CounterInitial extends CounterState {
  CounterInitial() : super(0);
}

class CounterUpdated extends CounterState {
  CounterUpdated(int count) : super(count);
}

// 3. Create Bloc
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterInitial()) {
    on<IncrementEvent>((event, emit) {
      emit(CounterUpdated(state.count + 1));
    });

    on<DecrementEvent>((event, emit) {
      emit(CounterUpdated(state.count - 1));
    });

    on<ResetEvent>((event, emit) {
      emit(CounterInitial());
    });
  }
}

// 4. Provide Bloc
void main() {
  runApp(
    BlocProvider(
      create: (context) => CounterBloc(),
      child: MyApp(),
    ),
  );
}

// 5. Consume with BlocBuilder
class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: BlocBuilder<CounterBloc, CounterState>(
        builder: (context, state) {
          return Center(
            child: Text('Count: ${state.count}'),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          context.read<CounterBloc>().add(IncrementEvent());
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### Advanced Example (Real-world)

```dart
// Events
abstract class UserEvent {}

class LoadUserEvent extends UserEvent {
  final int userId;
  LoadUserEvent(this.userId);
}

class UpdateUserEvent extends UserEvent {
  final User user;
  UpdateUserEvent(this.user);
}

class LogoutEvent extends UserEvent {}

// States
abstract class UserState {}

class UserInitial extends UserState {}

class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final User user;
  UserLoaded(this.user);
}

class UserError extends UserState {
  final String message;
  UserError(this.message);
}

// Bloc
class UserBloc extends Bloc<UserEvent, UserState> {
  final UserRepository _repository;

  UserBloc(this._repository) : super(UserInitial()) {
    on<LoadUserEvent>(_onLoadUser);
    on<UpdateUserEvent>(_onUpdateUser);
    on<LogoutEvent>(_onLogout);
  }

  Future<void> _onLoadUser(
    LoadUserEvent event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    try {
      final user = await _repository.getUser(event.userId);
      emit(UserLoaded(user));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }

  Future<void> _onUpdateUser(
    UpdateUserEvent event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    try {
      final user = await _repository.updateUser(event.user);
      emit(UserLoaded(user));
    } catch (e) {
      emit(UserError(e.toString()));
    }
  }

  void _onLogout(LogoutEvent event, Emitter<UserState> emit) {
    emit(UserInitial());
  }
}

// Usage with BlocConsumer (builder + listener)
class UserProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocConsumer<UserBloc, UserState>(
      listener: (context, state) {
        if (state is UserError) {
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(content: Text(state.message)),
          );
        }
      },
      builder: (context, state) {
        if (state is UserLoading) {
          return Center(child: CircularProgressIndicator());
        }

        if (state is UserLoaded) {
          return Column(
            children: [
              Text('Name: ${state.user.name}'),
              Text('Email: ${state.user.email}'),
              ElevatedButton(
                onPressed: () {
                  context.read<UserBloc>().add(LogoutEvent());
                },
                child: Text('Logout'),
              ),
            ],
          );
        }

        return Center(child: Text('No user'));
      },
    );
  }
}
```

**Pros:**
- ✅ Clear separation of concerns
- ✅ Predictable state changes
- ✅ Excellent for complex state
- ✅ Great testing support
- ✅ Time-travel debugging

**Cons:**
- ❌ Lots of boilerplate
- ❌ Steeper learning curve
- ❌ Overkill for simple apps

**When to use:** Large apps, complex state logic, team projects requiring strict patterns

---

## 5. GetX (All-in-one)

**Android Equivalent:** - (unique to Flutter)

```yaml
dependencies:
  get: ^4.6.6
```

### Basic Example

```dart
// 1. Create Controller
class CounterController extends GetxController {
  var count = 0.obs;  // Observable

  void increment() => count++;
  void decrement() => count--;
  void reset() => count.value = 0;
}

// 2. No provider needed!
void main() {
  runApp(MyApp());
}

// 3. Use Get.put to inject
class CounterScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final controller = Get.put(CounterController());

    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Obx(() => Text('Count: ${controller.count}')),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: controller.increment,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

### Advanced Example

```dart
// User controller
class UserController extends GetxController {
  final UserRepository _repository = Get.find();

  final Rx<User?> user = Rx<User?>(null);
  final RxBool isLoading = false.obs;
  final RxString error = ''.obs;

  @override
  void onInit() {
    super.onInit();
    loadUser(1);
  }

  Future<void> loadUser(int id) async {
    try {
      isLoading.value = true;
      error.value = '';
      user.value = await _repository.getUser(id);
    } catch (e) {
      error.value = e.toString();
    } finally {
      isLoading.value = false;
    }
  }

  void logout() {
    user.value = null;
  }
}

// Usage
class UserProfileScreen extends StatelessWidget {
  final controller = Get.find<UserController>();

  @override
  Widget build(BuildContext context) {
    return Obx(() {
      if (controller.isLoading.value) {
        return Center(child: CircularProgressIndicator());
      }

      if (controller.error.value.isNotEmpty) {
        return Center(child: Text('Error: ${controller.error.value}'));
      }

      final user = controller.user.value;
      if (user == null) {
        return Center(child: Text('No user'));
      }

      return Column(
        children: [
          Text('Name: ${user.name}'),
          Text('Email: ${user.email}'),
          ElevatedButton(
            onPressed: controller.logout,
            child: Text('Logout'),
          ),
        ],
      );
    });
  }
}

// Navigation (bonus feature)
Get.to(() => UserProfileScreen());  // Navigate
Get.back();  // Go back
Get.snackbar('Success', 'User updated');  // Show snackbar
```

**Pros:**
- ✅ Minimal boilerplate
- ✅ Very easy to learn
- ✅ Includes navigation, dialogs, etc.
- ✅ Fast development

**Cons:**
- ❌ Magic/"black box" behavior
- ❌ Global state can be messy
- ❌ Not recommended by Flutter team
- ❌ Can lead to bad practices

**When to use:** Rapid prototyping, simple apps, solo projects

---

## 6. MobX

**Android Equivalent:** MobX Android

```yaml
dependencies:
  mobx: ^2.3.0
  flutter_mobx: ^2.2.0

dev_dependencies:
  build_runner: ^2.4.7
  mobx_codegen: ^2.6.0
```

```dart
// Store
import 'package:mobx/mobx.dart';

part 'counter_store.g.dart';

class CounterStore = _CounterStore with _$CounterStore;

abstract class _CounterStore with Store {
  @observable
  int count = 0;

  @action
  void increment() {
    count++;
  }

  @action
  void decrement() {
    count--;
  }

  @computed
  bool get isEven => count % 2 == 0;
}

// Usage
class CounterScreen extends StatelessWidget {
  final store = CounterStore();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Observer(
          builder: (_) => Column(
            children: [
              Text('Count: ${store.count}'),
              Text('Is even: ${store.isEven}'),
            ],
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: store.increment,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

**Pros:**
- ✅ Reactive and transparent
- ✅ Minimal boilerplate
- ✅ Computed values

**Cons:**
- ❌ Requires code generation
- ❌ Smaller community in Flutter

**When to use:** If you love MobX from other platforms

---

## Comparison Matrix

| Feature | setState | Provider | Riverpod | BLoC | GetX | MobX |
|---------|----------|----------|----------|------|------|------|
| **Learning Curve** | Easy | Easy | Medium | Hard | Easy | Medium |
| **Boilerplate** | None | Low | Low | High | Minimal | Low |
| **Testability** | Poor | Good | Excellent | Excellent | Good | Good |
| **Scalability** | Poor | Good | Excellent | Excellent | Good | Good |
| **Documentation** | Excellent | Excellent | Good | Excellent | Good | Good |
| **Community** | ✅ | ✅ | ✅ | ✅ | ⚠️ | ⚠️ |
| **Flutter Team** | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Type Safety** | ✅ | ✅ | ✅✅ | ✅ | ⚠️ | ✅ |
| **Code Generation** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

## When to Use Each

### Use setState when:
- Simple local UI state
- Toggle buttons, checkboxes
- Form validation in a single screen
- Temporary UI state

### Use Provider when:
- Small to medium apps
- Learning Flutter
- Need simple DI
- Team prefers simplicity

### Use Riverpod when:
- Medium to large apps
- Want type safety
- Complex dependency graphs
- Need excellent testing

### Use BLoC when:
- Large enterprise apps
- Complex business logic
- Team needs strict patterns
- Time-travel debugging needed

### Use GetX when:
- Rapid prototyping
- Small projects
- Solo developer
- Want simplicity over best practices

### Use MobX when:
- Coming from React/MobX background
- Want reactive programming
- Comfortable with code generation

## Recommended Approach

**For most developers:**

1. **Start with Provider** - Easy to learn, official support
2. **Graduate to Riverpod** - When you need more features
3. **Consider BLoC** - For very large/complex apps

**Migration path:**
```
setState → Provider → Riverpod
                    ↘ BLoC (for complex apps)
```

## Real-world Example Comparison

Let's build a simple user profile screen with each approach:

**Requirement:** Load user data, show loading state, handle errors

See implementations in previous sections for full code examples.

## Best Practices (All Approaches)

1. **Keep state immutable** - Use `copyWith` or `freezed`
2. **Separate business logic** - Don't mix with UI
3. **Test your state** - Write unit tests
4. **Use const widgets** - Improve performance
5. **Dispose resources** - Cancel streams, timers
6. **Handle errors** - Always catch exceptions
7. **Loading states** - Show progress indicators
8. **Debounce user input** - For search, etc.

## Next Steps

Now that you understand state management, let's explore navigation and routing in [Section 8: Navigation & Routing](./08-navigation-routing.md).
