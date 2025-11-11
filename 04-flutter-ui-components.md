# Section 4: Flutter UI Components

## Introduction

In Flutter, **everything is a widget**. Unlike Android's XML layouts + Kotlin code separation, Flutter uses a declarative approach where UI is built entirely in Dart code using widget composition.

## Fundamental Concept: The Widget Tree

### Android Approach
```xml
<!-- XML Layout -->
<LinearLayout>
    <TextView />
    <Button />
</LinearLayout>
```
```kotlin
// Kotlin code
findViewById<Button>(R.id.button).setOnClickListener { }
```

### Flutter Approach
```dart
// Everything in Dart
Column(
  children: [
    Text('Hello'),
    ElevatedButton(
      onPressed: () { },
      child: Text('Click'),
    ),
  ],
)
```

## Widget Types

### 1. StatelessWidget vs StatefulWidget

**StatelessWidget** - Immutable, no internal state
```dart
class WelcomeScreen extends StatelessWidget {
  final String userName;

  const WelcomeScreen({Key? key, required this.userName}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Welcome')),
      body: Text('Hello, $userName!'),
    );
  }
}
```

**StatefulWidget** - Mutable state that can change
```dart
class Counter extends StatefulWidget {
  const Counter({Key? key}) : super(key: key);

  @override
  State<Counter> createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

## Major Layout Widgets

### 1. Container (Similar to FrameLayout/RelativeLayout)

**Android:**
```xml
<FrameLayout
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:background="@color/blue"
    android:padding="16dp"
    android:layout_margin="8dp">
    <TextView />
</FrameLayout>
```

**Flutter:**
```dart
Container(
  width: 200,
  height: 200,
  color: Colors.blue,
  padding: EdgeInsets.all(16),
  margin: EdgeInsets.all(8),
  child: Text('Content'),
)

// With decoration (for rounded corners, shadows, etc.)
Container(
  width: 200,
  height: 200,
  padding: EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(
        color: Colors.grey.withOpacity(0.5),
        spreadRadius: 2,
        blurRadius: 5,
        offset: Offset(0, 3),
      ),
    ],
  ),
  child: Text('Content'),
)
```

### 2. Row & Column (Similar to LinearLayout)

**Android (Horizontal):**
```xml
<LinearLayout
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <TextView />
    <Button />
    <ImageView />
</LinearLayout>
```

**Flutter (Row):**
```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,  // Similar to gravity
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Text('Label'),
    ElevatedButton(onPressed: () {}, child: Text('Button')),
    Icon(Icons.home),
  ],
)

// Column for vertical
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Text('Item 1'),
    Text('Item 2'),
    Text('Item 3'),
  ],
)
```

**MainAxisAlignment** (similar to LinearLayout gravity):
- `start` - Start of the axis
- `end` - End of the axis
- `center` - Center
- `spaceBetween` - Space between children
- `spaceAround` - Space around children
- `spaceEvenly` - Even space

**CrossAxisAlignment**:
- `start` - Start of cross axis
- `end` - End of cross axis
- `center` - Center
- `stretch` - Stretch to fill

### 3. Stack (Similar to FrameLayout with overlapping views)

**Android:**
```xml
<FrameLayout>
    <ImageView />  <!-- Background -->
    <TextView />   <!-- Overlay -->
</FrameLayout>
```

**Flutter:**
```dart
Stack(
  children: [
    // Background
    Container(
      width: 300,
      height: 300,
      color: Colors.blue,
    ),
    // Overlay - positioned
    Positioned(
      top: 50,
      left: 50,
      child: Text('Overlay'),
    ),
    // Overlay - aligned
    Align(
      alignment: Alignment.bottomRight,
      child: Icon(Icons.favorite),
    ),
  ],
)
```

### 4. Scaffold (Material Design Screen Structure)

```dart
Scaffold(
  appBar: AppBar(
    title: Text('My App'),
    actions: [
      IconButton(
        icon: Icon(Icons.search),
        onPressed: () {},
      ),
    ],
  ),
  body: Center(
    child: Text('Content'),
  ),
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: Icon(Icons.add),
  ),
  drawer: Drawer(
    child: ListView(
      children: [
        DrawerHeader(
          child: Text('Menu'),
        ),
        ListTile(
          title: Text('Item 1'),
          onTap: () {},
        ),
      ],
    ),
  ),
  bottomNavigationBar: BottomNavigationBar(
    items: [
      BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
      BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
    ],
  ),
)
```

### 5. ListView (Similar to RecyclerView)

**Android:**
```kotlin
recyclerView.apply {
    layoutManager = LinearLayoutManager(context)
    adapter = MyAdapter(items)
}
```

**Flutter:**

**Simple ListView:**
```dart
ListView(
  children: [
    ListTile(title: Text('Item 1')),
    ListTile(title: Text('Item 2')),
    ListTile(title: Text('Item 3')),
  ],
)
```

**ListView.builder (Efficient, like RecyclerView):**
```dart
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    final item = items[index];
    return ListTile(
      leading: CircleAvatar(child: Text('${index + 1}')),
      title: Text(item.title),
      subtitle: Text(item.subtitle),
      trailing: Icon(Icons.arrow_forward),
      onTap: () {
        // Handle tap
      },
    );
  },
)
```

**ListView.separated (With dividers):**
```dart
ListView.separated(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ListTile(title: Text(items[index]));
  },
  separatorBuilder: (context, index) {
    return Divider();  // Or custom separator
  },
)
```

**Horizontal ListView:**
```dart
ListView.builder(
  scrollDirection: Axis.horizontal,
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Container(
      width: 100,
      margin: EdgeInsets.all(8),
      child: Text(items[index]),
    );
  },
)
```

### 6. GridView (Similar to GridLayoutManager)

**Android:**
```kotlin
recyclerView.layoutManager = GridLayoutManager(context, 2)
```

**Flutter:**
```dart
GridView.builder(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,  // Number of columns
    crossAxisSpacing: 10,
    mainAxisSpacing: 10,
    childAspectRatio: 1.0,
  ),
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Card(
      child: Center(
        child: Text(items[index]),
      ),
    );
  },
)

// Or with max cross-axis extent
GridView.builder(
  gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 200,  // Max width of each item
    crossAxisSpacing: 10,
    mainAxisSpacing: 10,
  ),
  itemCount: items.length,
  itemBuilder: (context, index) {
    return Card(child: Text(items[index]));
  },
)
```

## Common UI Widgets

### 1. Text (TextView)

**Android:**
```xml
<TextView
    android:text="Hello World"
    android:textSize="18sp"
    android:textColor="@color/black"
    android:textStyle="bold" />
```

**Flutter:**
```dart
Text(
  'Hello World',
  style: TextStyle(
    fontSize: 18,
    color: Colors.black,
    fontWeight: FontWeight.bold,
    fontStyle: FontStyle.italic,
    decoration: TextDecoration.underline,
  ),
)

// Rich text (like SpannableString)
Text.rich(
  TextSpan(
    text: 'Hello ',
    style: TextStyle(fontSize: 16),
    children: [
      TextSpan(
        text: 'World',
        style: TextStyle(fontWeight: FontWeight.bold, color: Colors.blue),
      ),
    ],
  ),
)
```

### 2. Buttons

**Android:**
```xml
<Button />
<ImageButton />
<FloatingActionButton />
```

**Flutter:**

```dart
// Elevated Button (Material Design raised button)
ElevatedButton(
  onPressed: () {
    print('Pressed');
  },
  child: Text('Click Me'),
)

// Text Button (Flat button)
TextButton(
  onPressed: () {},
  child: Text('Click Me'),
)

// Outlined Button
OutlinedButton(
  onPressed: () {},
  child: Text('Click Me'),
)

// Icon Button
IconButton(
  icon: Icon(Icons.favorite),
  onPressed: () {},
)

// Floating Action Button
FloatingActionButton(
  onPressed: () {},
  child: Icon(Icons.add),
)

// Custom styled button
ElevatedButton(
  onPressed: () {},
  style: ElevatedButton.styleFrom(
    backgroundColor: Colors.blue,
    foregroundColor: Colors.white,
    padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(8),
    ),
  ),
  child: Text('Custom Button'),
)
```

### 3. TextField (EditText)

**Android:**
```xml
<EditText
    android:hint="Enter name"
    android:inputType="text" />
```

**Flutter:**
```dart
TextField(
  decoration: InputDecoration(
    labelText: 'Name',
    hintText: 'Enter your name',
    prefixIcon: Icon(Icons.person),
    suffixIcon: Icon(Icons.clear),
    border: OutlineInputBorder(),
  ),
  onChanged: (value) {
    print('Text changed: $value');
  },
)

// With controller (like EditText.getText())
final TextEditingController _controller = TextEditingController();

@override
void initState() {
  super.initState();
  _controller.text = 'Initial text';
  _controller.addListener(() {
    print('Current text: ${_controller.text}');
  });
}

@override
void dispose() {
  _controller.dispose();
  super.dispose();
}

TextField(
  controller: _controller,
  decoration: InputDecoration(labelText: 'Name'),
)

// Get text
String text = _controller.text;

// Set text
_controller.text = 'New text';
```

### 4. Image (ImageView)

**Android:**
```xml
<ImageView
    android:src="@drawable/image"
    android:scaleType="centerCrop" />
```

**Flutter:**
```dart
// From assets
Image.asset(
  'assets/images/logo.png',
  width: 100,
  height: 100,
  fit: BoxFit.cover,  // Similar to scaleType
)

// From network
Image.network(
  'https://example.com/image.jpg',
  width: 100,
  height: 100,
  fit: BoxFit.cover,
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator();
  },
  errorBuilder: (context, error, stackTrace) {
    return Icon(Icons.error);
  },
)

// From file
Image.file(File('/path/to/image.jpg'))

// BoxFit options (similar to ScaleType)
// - BoxFit.fill -> FIT_XY
// - BoxFit.contain -> FIT_CENTER
// - BoxFit.cover -> CENTER_CROP
// - BoxFit.fitWidth -> FIT_START (width)
// - BoxFit.fitHeight -> FIT_START (height)
// - BoxFit.none -> CENTER
// - BoxFit.scaleDown -> CENTER_INSIDE
```

### 5. Card (CardView)

**Android:**
```xml
<androidx.cardview.widget.CardView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">
    <TextView />
</androidx.cardview.widget.CardView>
```

**Flutter:**
```dart
Card(
  elevation: 4,
  shape: RoundedRectangleBorder(
    borderRadius: BorderRadius.circular(8),
  ),
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Column(
      children: [
        Text('Title', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
        SizedBox(height: 8),
        Text('Content'),
      ],
    ),
  ),
)
```

### 6. AppBar (Toolbar)

**Android:**
```xml
<androidx.appcompat.widget.Toolbar
    android:id="@+id/toolbar"
    app:title="My App" />
```

**Flutter:**
```dart
AppBar(
  title: Text('My App'),
  leading: IconButton(
    icon: Icon(Icons.menu),
    onPressed: () {},
  ),
  actions: [
    IconButton(
      icon: Icon(Icons.search),
      onPressed: () {},
    ),
    IconButton(
      icon: Icon(Icons.more_vert),
      onPressed: () {},
    ),
  ],
  backgroundColor: Colors.blue,
  elevation: 4,
)
```

## Layout Constraints

### Expanded & Flexible (Similar to layout_weight)

**Android:**
```xml
<LinearLayout android:orientation="horizontal">
    <View
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />
    <View
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="2" />
</LinearLayout>
```

**Flutter:**
```dart
Row(
  children: [
    Expanded(
      flex: 1,
      child: Container(color: Colors.red),
    ),
    Expanded(
      flex: 2,
      child: Container(color: Colors.blue),
    ),
  ],
)

// Flexible (can shrink but doesn't have to expand)
Row(
  children: [
    Flexible(
      flex: 1,
      child: Text('This can shrink if needed'),
    ),
    Flexible(
      flex: 2,
      child: Text('This too'),
    ),
  ],
)
```

### SizedBox (Fixed spacing)

```dart
// Fixed size
SizedBox(
  width: 100,
  height: 100,
  child: Container(color: Colors.blue),
)

// Spacing
Column(
  children: [
    Text('First'),
    SizedBox(height: 16),  // 16dp spacing
    Text('Second'),
  ],
)
```

### Padding & Margin

**Android:**
```xml
<View
    android:padding="16dp"
    android:layout_margin="8dp" />
```

**Flutter:**
```dart
// Padding
Padding(
  padding: EdgeInsets.all(16),
  child: Text('Content'),
)

// Different padding for each side
Padding(
  padding: EdgeInsets.only(left: 16, top: 8, right: 16, bottom: 8),
  child: Text('Content'),
)

// Symmetric padding
Padding(
  padding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
  child: Text('Content'),
)

// Margin (use Container)
Container(
  margin: EdgeInsets.all(8),
  child: Text('Content'),
)
```

## Scrolling Widgets

### 1. SingleChildScrollView (ScrollView)

**Android:**
```xml
<ScrollView>
    <LinearLayout>
        <!-- Content -->
    </LinearLayout>
</ScrollView>
```

**Flutter:**
```dart
SingleChildScrollView(
  child: Column(
    children: [
      // Content that might overflow
      Container(height: 200, color: Colors.red),
      Container(height: 200, color: Colors.blue),
      Container(height: 200, color: Colors.green),
    ],
  ),
)
```

### 2. RefreshIndicator (SwipeRefreshLayout)

**Android:**
```kotlin
swipeRefreshLayout.setOnRefreshListener {
    loadData()
}
```

**Flutter:**
```dart
RefreshIndicator(
  onRefresh: () async {
    await loadData();
  },
  child: ListView.builder(
    itemCount: items.length,
    itemBuilder: (context, index) {
      return ListTile(title: Text(items[index]));
    },
  ),
)
```

## Interactive Widgets

### 1. GestureDetector

```dart
GestureDetector(
  onTap: () {
    print('Tapped');
  },
  onDoubleTap: () {
    print('Double tapped');
  },
  onLongPress: () {
    print('Long pressed');
  },
  onPanUpdate: (details) {
    print('Dragging: ${details.delta}');
  },
  child: Container(
    width: 200,
    height: 200,
    color: Colors.blue,
    child: Center(child: Text('Touch me')),
  ),
)
```

### 2. InkWell (Ripple effect)

```dart
InkWell(
  onTap: () {
    print('Tapped');
  },
  child: Container(
    padding: EdgeInsets.all(16),
    child: Text('Tap for ripple'),
  ),
)
```

## Material Design vs Cupertino (iOS style)

Flutter provides two design systems:

### Material Design (Android-style)

```dart
import 'package:flutter/material.dart';

MaterialApp(
  home: Scaffold(
    appBar: AppBar(title: Text('Material')),
    body: Column(
      children: [
        ElevatedButton(onPressed: () {}, child: Text('Button')),
        CircularProgressIndicator(),
        Checkbox(value: true, onChanged: (val) {}),
        Switch(value: true, onChanged: (val) {}),
      ],
    ),
  ),
)
```

### Cupertino (iOS-style)

```dart
import 'package:flutter/cupertino.dart';

CupertinoApp(
  home: CupertinoPageScaffold(
    navigationBar: CupertinoNavigationBar(
      middle: Text('Cupertino'),
    ),
    child: Column(
      children: [
        CupertinoButton(onPressed: () {}, child: Text('Button')),
        CupertinoActivityIndicator(),
        CupertinoSwitch(value: true, onChanged: (val) {}),
      ],
    ),
  ),
)
```

### Platform-Adaptive Widgets

```dart
import 'dart:io' show Platform;

Widget buildButton() {
  if (Platform.isIOS) {
    return CupertinoButton(
      onPressed: () {},
      child: Text('Button'),
    );
  }
  return ElevatedButton(
    onPressed: () {},
    child: Text('Button'),
  );
}

// Or use platform-aware widgets
import 'package:flutter/foundation.dart';

final button = defaultTargetPlatform == TargetPlatform.iOS
    ? CupertinoButton(onPressed: () {}, child: Text('Button'))
    : ElevatedButton(onPressed: () {}, child: Text('Button'));
```

## Widget Comparison Table

| Android | Flutter Material | Flutter Cupertino |
|---------|------------------|-------------------|
| TextView | Text | Text |
| Button | ElevatedButton | CupertinoButton |
| EditText | TextField | CupertinoTextField |
| CheckBox | Checkbox | CupertinoSwitch |
| Switch | Switch | CupertinoSwitch |
| ProgressBar | CircularProgressIndicator | CupertinoActivityIndicator |
| AlertDialog | AlertDialog | CupertinoAlertDialog |
| Toolbar | AppBar | CupertinoNavigationBar |
| BottomSheet | showModalBottomSheet | CupertinoActionSheet |
| RecyclerView | ListView.builder | ListView.builder |
| GridView | GridView.builder | GridView.builder |

## Best Practices

### 1. Extract Widgets

```dart
// ❌ BAD - Everything in one build method
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: Column(
      children: [
        Container(
          padding: EdgeInsets.all(16),
          child: Row(
            children: [
              Icon(Icons.person),
              SizedBox(width: 8),
              Text('User Name'),
            ],
          ),
        ),
        // ... more complex UI
      ],
    ),
  );
}

// ✅ GOOD - Extract to methods or widgets
@override
Widget build(BuildContext context) {
  return Scaffold(
    body: Column(
      children: [
        _buildUserHeader(),
        _buildContent(),
      ],
    ),
  );
}

Widget _buildUserHeader() {
  return Container(
    padding: EdgeInsets.all(16),
    child: Row(
      children: [
        Icon(Icons.person),
        SizedBox(width: 8),
        Text('User Name'),
      ],
    ),
  );
}

// Or even better - separate widget
class UserHeader extends StatelessWidget {
  final String userName;

  const UserHeader({Key? key, required this.userName}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      child: Row(
        children: [
          Icon(Icons.person),
          SizedBox(width: 8),
          Text(userName),
        ],
      ),
    );
  }
}
```

### 2. Use const Constructors

```dart
// ✅ GOOD - const for performance
const Text('Hello')
const SizedBox(height: 16)
const Icon(Icons.home)

// Entire widget trees can be const
const Column(
  children: [
    Text('Title'),
    SizedBox(height: 8),
    Text('Content'),
  ],
)
```

### 3. Avoid Deep Nesting

```dart
// ❌ BAD - Too deep
Container(
  child: Center(
    child: Column(
      children: [
        Container(
          child: Row(
            children: [
              Container(
                child: Text('Deep'),
              ),
            ],
          ),
        ),
      ],
    ),
  ),
)

// ✅ GOOD - Extract and flatten
_buildContent()
```

## Next Steps

Now that you understand Flutter UI components, let's explore popular libraries and packages in [Section 5: Popular Libraries & Packages](./05-popular-libraries.md).
