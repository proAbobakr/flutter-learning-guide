# Section 8: Navigation & Routing

## Introduction

Navigation in Flutter works differently from Android's Activity/Fragment navigation. This section covers Flutter's navigation approaches and compares them with Android's Navigation Component.

## Android Navigation Review

**Android (Navigation Component):**
```kotlin
// Navigate to destination
findNavController().navigate(R.id.userProfileFragment)

// Navigate with arguments
val args = Bundle().apply {
    putInt("userId", 123)
}
findNavController().navigate(R.id.userProfileFragment, args)

// Navigate back
findNavController().navigateUp()

// Deep linking
<nav-graph>
    <deepLink app:uri="myapp://user/{userId}" />
</nav-graph>
```

## Flutter Navigation Approaches

### 1. Navigator 1.0 (Basic, Imperative)

**Android Equivalent:** Manual Fragment transactions

#### Basic Navigation

```dart
// Push new screen
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => UserScreen()),
);

// Pop current screen
Navigator.pop(context);

// Pop with result
Navigator.pop(context, result);

// Push and wait for result
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => UserScreen()),
);
```

#### Named Routes

```dart
// Define routes in MaterialApp
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      initialRoute: '/',
      routes: {
        '/': (context) => HomeScreen(),
        '/user': (context) => UserScreen(),
        '/settings': (context) => SettingsScreen(),
      },
    );
  }
}

// Navigate using named routes
Navigator.pushNamed(context, '/user');

// With arguments
Navigator.pushNamed(
  context,
  '/user',
  arguments: {'userId': 123},
);

// Retrieve arguments
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as Map;
    final userId = args['userId'];

    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
    );
  }
}
```

#### Advanced Navigation Operations

```dart
// Push and remove all previous routes
Navigator.pushNamedAndRemoveUntil(
  context,
  '/home',
  (route) => false,  // Remove all
);

// Replace current route
Navigator.pushReplacementNamed(context, '/login');

// Pop until specific route
Navigator.popUntil(context, ModalRoute.withName('/home'));

// Can pop? (check if can go back)
if (Navigator.canPop(context)) {
  Navigator.pop(context);
}
```

#### Custom Transitions

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => UserScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      const begin = Offset(1.0, 0.0);
      const end = Offset.zero;
      const curve = Curves.ease;

      var tween = Tween(begin: begin, end: end).chain(
        CurveTween(curve: curve),
      );

      return SlideTransition(
        position: animation.drive(tween),
        child: child,
      );
    },
  ),
);
```

---

### 2. Navigator 2.0 (Declarative)

**Android Equivalent:** Navigation Component with NavGraph

More complex but provides:
- Deep linking support
- Web URL support
- Declarative routing
- Better state management

```dart
// App structure
class MyApp extends StatefulWidget {
  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String? _selectedUserId;

  void _selectUser(String userId) {
    setState(() {
      _selectedUserId = userId;
    });
  }

  void _clearSelection() {
    setState(() {
      _selectedUserId = null;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Navigator(
        pages: [
          MaterialPage(
            key: ValueKey('HomePage'),
            child: HomeScreen(onSelectUser: _selectUser),
          ),
          if (_selectedUserId != null)
            MaterialPage(
              key: ValueKey('UserPage'),
              child: UserScreen(
                userId: _selectedUserId!,
                onBack: _clearSelection,
              ),
            ),
        ],
        onPopPage: (route, result) {
          if (!route.didPop(result)) {
            return false;
          }
          _clearSelection();
          return true;
        },
      ),
    );
  }
}
```

---

### 3. go_router (Recommended)

**Android Equivalent:** Navigation Component

```yaml
dependencies:
  go_router: ^13.0.0
```

#### Basic Setup

```dart
import 'package:go_router/go_router.dart';

// Define routes
final GoRouter _router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
    GoRoute(
      path: '/user/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return UserScreen(userId: id);
      },
    ),
    GoRoute(
      path: '/settings',
      builder: (context, state) => SettingsScreen(),
      routes: [
        // Nested route: /settings/profile
        GoRoute(
          path: 'profile',
          builder: (context, state) => ProfileSettingsScreen(),
        ),
      ],
    ),
  ],
  errorBuilder: (context, state) => ErrorScreen(error: state.error),
);

// Use in app
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
    );
  }
}

// Navigate
context.go('/user/123');  // Replace
context.push('/settings');  // Push
context.pop();  // Pop
```

#### Advanced Features

```dart
// Query parameters
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'] ?? '';
    return SearchScreen(query: query);
  },
)

// Navigate: context.go('/search?q=flutter')

// Redirect/Guards
final router = GoRouter(
  redirect: (context, state) {
    final loggedIn = // check auth state
    final goingToLogin = state.matchedLocation == '/login';

    if (!loggedIn && !goingToLogin) {
      return '/login';
    }
    if (loggedIn && goingToLogin) {
      return '/';
    }
    return null;  // No redirect
  },
  routes: [...],
);

// Refresh on state change
GoRouter.optionURLReflectsImperativeAPIs = true;

final router = GoRouter(
  refreshListenable: authNotifier,  // Refresh when auth changes
  routes: [...],
);

// Shell routes (persistent UI)
GoRoute(
  path: '/',
  builder: (context, state, child) {
    return ScaffoldWithNavBar(child: child);
  },
  routes: [
    GoRoute(path: 'home', builder: (context, state) => HomeScreen()),
    GoRoute(path: 'profile', builder: (context, state) => ProfileScreen()),
  ],
);
```

#### Type-safe Routes

```dart
// Define routes with type safety
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) => HomeScreen();
}

class UserRoute extends GoRouteData {
  final int userId;
  const UserRoute({required this.userId});

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return UserScreen(userId: userId);
  }
}

// Navigate
const HomeRoute().go(context);
UserRoute(userId: 123).push(context);
```

---

### 4. auto_route (Code Generation)

**Android Equivalent:** SafeArgs

```yaml
dependencies:
  auto_route: ^7.8.4

dev_dependencies:
  auto_route_generator: ^7.3.2
  build_runner: ^2.4.7
```

```dart
// Define routes
import 'package:auto_route/auto_route.dart';

@AutoRouterConfig()
class AppRouter extends $AppRouter {
  @override
  List<AutoRoute> get routes => [
    AutoRoute(page: HomeRoute.page, initial: true),
    AutoRoute(page: UserRoute.page),
    AutoRoute(page: SettingsRoute.page),
  ];
}

// Screen with routing annotation
@RoutePage()
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: ElevatedButton(
        onPressed: () {
          context.router.push(UserRoute(userId: 123));
        },
        child: Text('Go to User'),
      ),
    );
  }
}

@RoutePage()
class UserScreen extends StatelessWidget {
  final int userId;

  const UserScreen({@PathParam('id') required this.userId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
    );
  }
}

// Use in app
void main() {
  final appRouter = AppRouter();
  runApp(
    MaterialApp.router(
      routerConfig: appRouter.config(),
    ),
  );
}
```

---

## Passing Data Between Screens

### Android Approach

```kotlin
// Send data
val intent = Intent(this, UserActivity::class.java).apply {
    putExtra("userId", 123)
}
startActivity(intent)

// Receive data
val userId = intent.getIntExtra("userId", -1)

// Result
startActivityForResult(intent, REQUEST_CODE)
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    if (requestCode == REQUEST_CODE && resultCode == RESULT_OK) {
        val result = data?.getStringExtra("result")
    }
}
```

### Flutter Approach

#### Forward Navigation

```dart
// Send data via constructor
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => UserScreen(userId: 123),
  ),
);

class UserScreen extends StatelessWidget {
  final int userId;

  const UserScreen({required this.userId});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('User $userId')),
    );
  }
}
```

#### Return Data

```dart
// Screen 1: Push and wait for result
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SelectColorScreen()),
);

if (result != null) {
  print('Selected color: $result');
}

// Screen 2: Pop with result
class SelectColorScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () => Navigator.pop(context, 'Red'),
            child: Text('Red'),
          ),
          ElevatedButton(
            onPressed: () => Navigator.pop(context, 'Blue'),
            child: Text('Blue'),
          ),
        ],
      ),
    );
  }
}
```

---

## Bottom Navigation & Tabs

### Android (BottomNavigationView)

```kotlin
bottomNavigationView.setOnItemSelectedListener { item ->
    when (item.itemId) {
        R.id.nav_home -> showFragment(HomeFragment())
        R.id.nav_profile -> showFragment(ProfileFragment())
    }
    true
}
```

### Flutter

```dart
class MainScreen extends StatefulWidget {
  @override
  State<MainScreen> createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  int _currentIndex = 0;

  final List<Widget> _screens = [
    HomeScreen(),
    SearchScreen(),
    ProfileScreen(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _screens[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        items: [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'Home',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'Search',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'Profile',
          ),
        ],
      ),
    );
  }
}
```

### With go_router (Persistent State)

```dart
class ScaffoldWithNavBar extends StatelessWidget {
  final Widget child;

  const ScaffoldWithNavBar({required this.child});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).uri.path;
    if (location.startsWith('/home')) return 0;
    if (location.startsWith('/search')) return 1;
    if (location.startsWith('/profile')) return 2;
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/home');
        break;
      case 1:
        context.go('/search');
        break;
      case 2:
        context.go('/profile');
        break;
    }
  }
}
```

---

## Deep Linking

### Android

```xml
<!-- AndroidManifest.xml -->
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data
        android:scheme="https"
        android:host="myapp.com"
        android:pathPrefix="/user" />
</intent-filter>
```

### Flutter with go_router

**AndroidManifest.xml:**
```xml
<intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" />
    <data android:scheme="https" android:host="myapp.com" />
</intent-filter>
```

**Go Router:**
```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/user/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return UserScreen(userId: id);
      },
    ),
  ],
);

// Deep link automatically handled:
// myapp://user/123
// https://myapp.com/user/123
```

---

## Dialogs & Bottom Sheets

### Dialogs

```dart
// Show dialog
showDialog(
  context: context,
  builder: (context) {
    return AlertDialog(
      title: Text('Confirm'),
      content: Text('Are you sure?'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: Text('Cancel'),
        ),
        TextButton(
          onPressed: () => Navigator.pop(context, true),
          child: Text('OK'),
        ),
      ],
    );
  },
);

// Wait for result
final confirmed = await showDialog<bool>(
  context: context,
  builder: (context) => ConfirmDialog(),
);
```

### Bottom Sheets

```dart
// Modal bottom sheet
showModalBottomSheet(
  context: context,
  builder: (context) {
    return Container(
      height: 200,
      child: Column(
        children: [
          ListTile(
            leading: Icon(Icons.share),
            title: Text('Share'),
            onTap: () => Navigator.pop(context, 'share'),
          ),
          ListTile(
            leading: Icon(Icons.delete),
            title: Text('Delete'),
            onTap: () => Navigator.pop(context, 'delete'),
          ),
        ],
      ),
    );
  },
);
```

---

## Navigation Comparison

| Android | Flutter (Navigator 1.0) | Flutter (go_router) |
|---------|------------------------|---------------------|
| `startActivity()` | `Navigator.push()` | `context.go()` |
| `finish()` | `Navigator.pop()` | `context.pop()` |
| `startActivityForResult()` | `await Navigator.push()` | Query params / state |
| Deep linking | Intent filters | Automatic |
| BackStack | Navigator stack | Router state |
| SafeArgs | Manual types | Path parameters |

---

## Best Practices

1. **Use go_router for new apps** - Best for most use cases
2. **Named routes** - More maintainable than anonymous routes
3. **Type safety** - Use typed route parameters
4. **Handle back button** - Use `WillPopScope` for confirmation
5. **Deep linking** - Set up from the start
6. **Navigation service** - For navigation outside widgets
7. **Route guards** - Implement authentication checks
8. **Persistent state** - For bottom nav, use shell routes

### Navigation Service (for non-widget contexts)

```dart
class NavigationService {
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

  Future<dynamic> navigateTo(String routeName, {dynamic arguments}) {
    return navigatorKey.currentState!.pushNamed(routeName, arguments: arguments);
  }

  void goBack() {
    return navigatorKey.currentState!.pop();
  }
}

// In MaterialApp
final navigationService = NavigationService();

MaterialApp(
  navigatorKey: navigationService.navigatorKey,
  routes: {...},
);

// Use anywhere
navigationService.navigateTo('/home');
```

---

## Recommended Approach

**For most developers:**

1. **Start simple**: Navigator 1.0 with named routes
2. **Scale up**: Move to go_router as app grows
3. **Enterprise**: Consider auto_route for type safety

---

## Next Steps

Now that you understand navigation, let's explore best practices and tips in [Section 9: Best Practices & Tips](./09-best-practices.md).
