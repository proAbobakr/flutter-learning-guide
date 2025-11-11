# Section 3: App Lifecycle

## Introduction

Understanding lifecycle is crucial for proper resource management, state handling, and responding to system events. Flutter's lifecycle model is simpler than Android's Activity/Fragment lifecycle but just as powerful.

## Flutter Lifecycle Overview

Flutter has **two main lifecycle concepts**:
1. **App Lifecycle** - The entire application state
2. **Widget Lifecycle** - Individual widget state (StatefulWidget)

## 1. App Lifecycle (Application Level)

### Android App Lifecycle

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // App created
    }

    override fun onTerminate() {
        super.onTerminate()
        // App terminated
    }
}

// Activity lifecycle
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) { }
    override fun onStart() { }
    override fun onResume() { }
    override fun onPause() { }
    override fun onStop() { }
    override fun onDestroy() { }
}
```

### Flutter App Lifecycle

```dart
import 'package:flutter/material.dart';

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    // Register lifecycle observer
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    // Unregister lifecycle observer
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    super.didChangeAppLifecycleState(state);

    switch (state) {
      case AppLifecycleState.resumed:
        // App is visible and responding to user input
        // Similar to onResume()
        print('App resumed');
        break;

      case AppLifecycleState.inactive:
        // App is inactive (e.g., phone call, system dialog)
        // Similar to onPause()
        print('App inactive');
        break;

      case AppLifecycleState.paused:
        // App is not visible to user
        // Similar to onStop()
        print('App paused');
        break;

      case AppLifecycleState.detached:
        // App is still hosted on a flutter engine but detached
        // About to exit
        print('App detached');
        break;

      case AppLifecycleState.hidden:
        // App is hidden (added in Flutter 3.13)
        print('App hidden');
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: HomeScreen(),
    );
  }
}
```

### Lifecycle State Comparison

| Android | Flutter | Description |
|---------|---------|-------------|
| `onCreate()` | `AppLifecycleState.resumed` (first time) | App starts |
| `onResume()` | `AppLifecycleState.resumed` | App visible and active |
| `onPause()` | `AppLifecycleState.inactive` | Partially visible |
| `onStop()` | `AppLifecycleState.paused` | Not visible |
| `onDestroy()` | `AppLifecycleState.detached` | About to exit |

## 2. Widget Lifecycle (StatefulWidget)

### The Complete Widget Lifecycle

```dart
class LifecycleDemo extends StatefulWidget {
  const LifecycleDemo({Key? key}) : super(key: key);

  @override
  State<LifecycleDemo> createState() {
    print('1. createState() - Creates the State object');
    return _LifecycleDemoState();
  }
}

class _LifecycleDemoState extends State<LifecycleDemo> {
  int _counter = 0;

  // Constructor (rarely used)
  _LifecycleDemoState() {
    print('2. Constructor - State object created');
  }

  @override
  void initState() {
    super.initState();
    print('3. initState() - Called once when State object is inserted into tree');

    // Similar to Activity onCreate()
    // Use for:
    // - Initialize state variables
    // - Subscribe to streams
    // - Initialize animations
    // - Fetch initial data

    _loadInitialData();
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print('4. didChangeDependencies() - Called after initState and when InheritedWidget changes');

    // Use for:
    // - Access BuildContext-dependent data
    // - Theme, MediaQuery, InheritedWidget access
  }

  @override
  Widget build(BuildContext context) {
    print('5. build() - Called frequently, must be fast and pure');

    // Similar to Activity/Fragment onCreateView()
    // Returns the widget tree
    return Scaffold(
      appBar: AppBar(title: Text('Lifecycle Demo')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Counter: $_counter'),
            ElevatedButton(
              onPressed: () {
                setState(() {
                  _counter++;
                });
              },
              child: Text('Increment'),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void didUpdateWidget(LifecycleDemo oldWidget) {
    super.didUpdateWidget(oldWidget);
    print('6. didUpdateWidget() - Called when widget configuration changes');

    // Use when parent rebuilds with new configuration
    // Compare oldWidget with widget to detect changes
    if (oldWidget.key != widget.key) {
      // Handle configuration change
    }
  }

  @override
  void setState(VoidCallback fn) {
    print('7. setState() - Triggers rebuild');
    super.setState(fn);
  }

  @override
  void deactivate() {
    print('8. deactivate() - Called when State is removed from tree temporarily');
    super.deactivate();

    // Rarely used, useful for:
    // - Handling navigation changes
  }

  @override
  void dispose() {
    print('9. dispose() - Called when State is removed permanently');

    // Similar to Activity onDestroy()
    // Use for:
    // - Cancel subscriptions
    // - Dispose controllers
    // - Clean up resources

    _disposeResources();

    super.dispose();
  }

  Future<void> _loadInitialData() async {
    // Fetch data
  }

  void _disposeResources() {
    // Clean up
  }
}
```

### Lifecycle Flow Diagram

```
┌─────────────────────────────────────────────────────┐
│                   Constructor                        │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│                 createState()                        │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│                  initState()                         │
│         (Initialize state, subscribe streams)        │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│            didChangeDependencies()                   │
│         (Access inherited widgets)                   │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│                   build()                            │
│              (Build widget tree)                     │
└────────────────────┬────────────────────────────────┘
                     │
              ┌──────┴──────┐
              │             │
      ┌───────▼──────┐  ┌──▼─────────────┐
      │  setState()  │  │ Widget Config  │
      │   called     │  │    changes     │
      └───────┬──────┘  └──┬─────────────┘
              │             │
              │    ┌────────▼──────────────┐
              │    │ didUpdateWidget()     │
              │    └────────┬──────────────┘
              │             │
              └─────────────▼
                      build()
                        │
                 ┌──────┴──────┐
                 │             │
         ┌───────▼──────┐  ┌──▼───────────┐
         │ deactivate() │  │   dispose()  │
         │  (temporary) │  │ (permanent)  │
         └──────────────┘  └──────────────┘
```

## Comparison: Android Activity vs Flutter StatefulWidget

```kotlin
// Android Activity
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: MyViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // Initialize
    }

    override fun onStart() {
        super.onStart()
        // Subscribe to updates
    }

    override fun onResume() {
        super.onResume()
        // Start animations, video playback
    }

    override fun onPause() {
        super.onPause()
        // Pause animations, video playback
    }

    override fun onStop() {
        super.onStop()
        // Unsubscribe from updates
    }

    override fun onDestroy() {
        super.onDestroy()
        // Clean up resources
    }
}
```

```dart
// Flutter StatefulWidget
class HomeScreen extends StatefulWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> with WidgetsBindingObserver {
  late AnimationController _controller;
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    // Similar to onCreate()
    _controller = AnimationController(vsync: this);

    // Register app lifecycle observer
    WidgetsBinding.instance.addObserver(this);

    // Subscribe to streams
    _subscription = dataStream.listen((data) {
      setState(() {
        // Update state
      });
    });
  }

  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    // Similar to onResume/onPause
    if (state == AppLifecycleState.resumed) {
      _controller.forward();
    } else if (state == AppLifecycleState.paused) {
      _controller.stop();
    }
  }

  @override
  void dispose() {
    // Similar to onDestroy()
    _controller.dispose();
    _subscription.cancel();
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // Similar to onCreateView() but called frequently
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Container(),
    );
  }
}
```

## Common Use Cases

### 1. Loading Data on Start

**Android:**
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    viewModel.loadData()
}
```

**Flutter:**
```dart
@override
void initState() {
  super.initState();
  _loadData();
}

Future<void> _loadData() async {
  final data = await repository.fetchData();
  setState(() {
    _data = data;
  });
}
```

### 2. Subscribing to Streams/LiveData

**Android:**
```kotlin
override fun onStart() {
    super.onStart()
    viewModel.dataFlow.collect { data ->
        updateUI(data)
    }
}
```

**Flutter:**
```dart
late StreamSubscription _subscription;

@override
void initState() {
  super.initState();
  _subscription = dataStream.listen((data) {
    setState(() {
      _data = data;
    });
  });
}

@override
void dispose() {
  _subscription.cancel();
  super.dispose();
}
```

### 3. Managing Controllers (Animation, Text, Scroll)

**Android:**
```kotlin
private lateinit var animator: ValueAnimator

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    animator = ValueAnimator.ofFloat(0f, 1f)
}

override fun onDestroy() {
    animator.cancel()
    super.onDestroy()
}
```

**Flutter:**
```dart
class AnimatedScreen extends StatefulWidget {
  @override
  State<AnimatedScreen> createState() => _AnimatedScreenState();
}

class _AnimatedScreenState extends State<AnimatedScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(seconds: 2),
      vsync: this,
    );
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();  // Must dispose!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _controller,
      child: Container(),
    );
  }
}
```

### 4. Responding to Configuration Changes

**Android:**
```kotlin
override fun onConfigurationChanged(newConfig: Configuration) {
    super.onConfigurationChanged(newConfig)
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        // Handle landscape
    }
}
```

**Flutter:**
```dart
@override
Widget build(BuildContext context) {
  // Build is called on configuration changes automatically
  final orientation = MediaQuery.of(context).orientation;

  if (orientation == Orientation.landscape) {
    return buildLandscapeLayout();
  }
  return buildPortraitLayout();
}
```

## StatelessWidget vs StatefulWidget

### When to Use StatelessWidget

```dart
// Use when widget doesn't need to maintain state
class WelcomeText extends StatelessWidget {
  final String name;

  const WelcomeText({Key? key, required this.name}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text('Welcome, $name!');
  }
}
```

**Use StatelessWidget when:**
- Widget only depends on configuration (constructor parameters)
- No mutable state
- Similar to Android View with fixed properties

### When to Use StatefulWidget

```dart
// Use when widget needs to maintain and update state
class Counter extends StatefulWidget {
  const Counter({Key? key}) : super(key: key);

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _count++;
            });
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

**Use StatefulWidget when:**
- Widget needs to maintain mutable state
- Needs lifecycle methods (initState, dispose)
- Manages animations, controllers, or subscriptions
- Similar to Android Fragment with ViewModel

## Best Practices

### 1. Always Dispose Resources

```dart
class ResourceScreen extends StatefulWidget {
  @override
  State<ResourceScreen> createState() => _ResourceScreenState();
}

class _ResourceScreenState extends State<ResourceScreen> {
  late TextEditingController _textController;
  late AnimationController _animController;
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _textController = TextEditingController();
    _animController = AnimationController(vsync: this);
    _subscription = stream.listen((data) {});
  }

  @override
  void dispose() {
    // Dispose in reverse order of creation
    _subscription.cancel();
    _animController.dispose();
    _textController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

### 2. Avoid Heavy Work in build()

```dart
// ❌ BAD - Heavy computation in build
@override
Widget build(BuildContext context) {
  final processedData = expensiveComputation(rawData);  // Runs on every rebuild!
  return Text(processedData);
}

// ✅ GOOD - Compute once in initState or setState
late String _processedData;

@override
void initState() {
  super.initState();
  _processedData = expensiveComputation(rawData);
}

@override
Widget build(BuildContext context) {
  return Text(_processedData);
}
```

### 3. Use didChangeDependencies Carefully

```dart
@override
void didChangeDependencies() {
  super.didChangeDependencies();

  // ✅ GOOD - Access inherited widgets
  final theme = Theme.of(context);
  final mediaQuery = MediaQuery.of(context);

  // ❌ BAD - Don't do async work here (called multiple times)
  // _loadData();  // Wrong!
}
```

### 4. Handle Async Operations Safely

```dart
@override
void initState() {
  super.initState();
  _loadData();
}

Future<void> _loadData() async {
  final data = await repository.fetchData();

  // ✅ Check if widget is still mounted before calling setState
  if (mounted) {
    setState(() {
      _data = data;
    });
  }
}

@override
void dispose() {
  // Widget is being disposed
  super.dispose();
}
```

## Common Pitfalls

### 1. Forgetting to Call super

```dart
// ❌ WRONG
@override
void initState() {
  _loadData();
  super.initState();  // Should be first!
}

// ✅ CORRECT
@override
void initState() {
  super.initState();  // Always call first
  _loadData();
}

// ❌ WRONG
@override
void dispose() {
  super.dispose();  // Should be last!
  _controller.dispose();
}

// ✅ CORRECT
@override
void dispose() {
  _controller.dispose();
  super.dispose();  // Always call last
}
```

### 2. Not Disposing Controllers

```dart
// ❌ Memory leak!
class BadWidget extends StatefulWidget {
  @override
  State<BadWidget> createState() => _BadWidgetState();
}

class _BadWidgetState extends State<BadWidget> {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  // Missing dispose() - Memory leak!
}
```

### 3. Using BuildContext After dispose

```dart
// ❌ BAD
Future<void> _loadData() async {
  final data = await repository.fetchData();
  // Widget might be disposed by now!
  setState(() {
    _data = data;
  });
}

// ✅ GOOD
Future<void> _loadData() async {
  final data = await repository.fetchData();
  if (mounted) {  // Check if still in tree
    setState(() {
      _data = data;
    });
  }
}
```

## Summary

| Concept | Android | Flutter |
|---------|---------|---------|
| App-level lifecycle | Activity/Fragment | WidgetsBindingObserver |
| Component lifecycle | onCreate/onDestroy | initState/dispose |
| Update UI | invalidate() / ViewModel | setState() |
| Configuration change | onConfigurationChanged | build() called automatically |
| Background/Foreground | onPause/onResume | AppLifecycleState |
| Resource cleanup | onDestroy | dispose |

## Next Steps

Now that you understand lifecycle management, let's explore Flutter UI components in [Section 4: Flutter UI Components](./04-flutter-ui-components.md).
