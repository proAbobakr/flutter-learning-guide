# React Native UI Components

## Component Paradigm

### Android Views vs React Components

**Android (XML + Kotlin):**
```xml
<!-- layout.xml -->
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <TextView
        android:id="@+id/titleText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello" />

    <Button
        android:id="@+id/submitButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Submit" />
</LinearLayout>
```

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        findViewById<Button>(R.id.submitButton).setOnClickListener {
            // Handle click
        }
    }
}
```

**React Native (JSX + TypeScript):**
```typescript
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

const MyScreen: React.FC = () => {
    const [count, setCount] = useState(0);

    const handleSubmit = () => {
        setCount(count + 1);
    };

    return (
        <View style={styles.container}>
            <Text style={styles.title}>Hello</Text>
            <TouchableOpacity style={styles.button} onPress={handleSubmit}>
                <Text>Submit ({count})</Text>
            </TouchableOpacity>
        </View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 16,
    },
    title: {
        fontSize: 20,
        fontWeight: 'bold',
    },
    button: {
        padding: 12,
        backgroundColor: '#007AFF',
        borderRadius: 8,
        marginTop: 16,
    },
});

export default MyScreen;
```

## Core Components

### View (Like LinearLayout/FrameLayout)

```typescript
import { View, StyleSheet } from 'react-native';

const MyView = () => {
    return (
        <View style={styles.container}>
            {/* Child components */}
        </View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,                    // Takes all available space
        flexDirection: 'column',     // Like android:orientation
        justifyContent: 'center',    // Vertical alignment
        alignItems: 'center',        // Horizontal alignment
        padding: 16,
        backgroundColor: '#ffffff',
    },
});
```

### Text (Like TextView)

```typescript
import { Text, StyleSheet } from 'react-native';

const MyText = () => {
    return (
        <>
            <Text style={styles.title}>Hello World</Text>
            <Text style={styles.body}>
                This is a longer text with{' '}
                <Text style={styles.bold}>bold</Text> and{' '}
                <Text style={styles.link}>clickable</Text> parts.
            </Text>
        </>
    );
};

const styles = StyleSheet.create({
    title: {
        fontSize: 24,
        fontWeight: 'bold',
        color: '#000000',
    },
    body: {
        fontSize: 16,
        lineHeight: 24,
        color: '#333333',
    },
    bold: {
        fontWeight: 'bold',
    },
    link: {
        color: '#007AFF',
        textDecorationLine: 'underline',
    },
});
```

### TextInput (Like EditText)

```typescript
import { useState } from 'react';
import { TextInput, StyleSheet } from 'react-native';

const MyInput = () => {
    const [text, setText] = useState('');
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    return (
        <>
            {/* Basic input */}
            <TextInput
                style={styles.input}
                value={text}
                onChangeText={setText}
                placeholder="Enter text"
            />

            {/* Email input */}
            <TextInput
                style={styles.input}
                value={email}
                onChangeText={setEmail}
                placeholder="Enter email"
                keyboardType="email-address"
                autoCapitalize="none"
                autoCorrect={false}
            />

            {/* Password input */}
            <TextInput
                style={styles.input}
                value={password}
                onChangeText={setPassword}
                placeholder="Enter password"
                secureTextEntry
            />
        </>
    );
};

const styles = StyleSheet.create({
    input: {
        height: 48,
        borderWidth: 1,
        borderColor: '#cccccc',
        borderRadius: 8,
        paddingHorizontal: 12,
        fontSize: 16,
        marginVertical: 8,
    },
});
```

### Buttons

#### TouchableOpacity (Most Common)

```typescript
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

const MyButton = () => {
    const handlePress = () => {
        console.log('Button pressed');
    };

    return (
        <TouchableOpacity
            style={styles.button}
            onPress={handlePress}
            activeOpacity={0.7}  // Opacity when pressed
        >
            <Text style={styles.buttonText}>Press Me</Text>
        </TouchableOpacity>
    );
};

const styles = StyleSheet.create({
    button: {
        backgroundColor: '#007AFF',
        padding: 16,
        borderRadius: 8,
        alignItems: 'center',
    },
    buttonText: {
        color: '#ffffff',
        fontSize: 16,
        fontWeight: '600',
    },
});
```

#### Pressable (Modern Alternative)

```typescript
import { Pressable, Text, StyleSheet } from 'react-native';

const MyPressable = () => {
    return (
        <Pressable
            style={({ pressed }) => [
                styles.button,
                pressed && styles.buttonPressed
            ]}
            onPress={() => console.log('Pressed')}
            onLongPress={() => console.log('Long pressed')}
        >
            {({ pressed }) => (
                <Text style={styles.buttonText}>
                    {pressed ? 'Pressed!' : 'Press Me'}
                </Text>
            )}
        </Pressable>
    );
};

const styles = StyleSheet.create({
    button: {
        backgroundColor: '#007AFF',
        padding: 16,
        borderRadius: 8,
        alignItems: 'center',
    },
    buttonPressed: {
        backgroundColor: '#0051D5',
    },
    buttonText: {
        color: '#ffffff',
        fontSize: 16,
        fontWeight: '600',
    },
});
```

### Image (Like ImageView)

```typescript
import { Image, StyleSheet } from 'react-native';

const MyImage = () => {
    return (
        <>
            {/* Local image */}
            <Image
                source={require('../assets/logo.png')}
                style={styles.image}
            />

            {/* Remote image */}
            <Image
                source={{ uri: 'https://example.com/image.jpg' }}
                style={styles.image}
                resizeMode="cover"  // cover, contain, stretch, repeat, center
            />

            {/* With loading */}
            <Image
                source={{ uri: 'https://example.com/image.jpg' }}
                style={styles.image}
                loadingIndicatorSource={require('../assets/loading.gif')}
            />
        </>
    );
};

const styles = StyleSheet.create({
    image: {
        width: 200,
        height: 200,
        borderRadius: 8,
    },
});
```

### ScrollView (Like ScrollView)

```typescript
import { ScrollView, View, Text, StyleSheet } from 'react-native';

const MyScrollView = () => {
    return (
        <ScrollView
            style={styles.container}
            contentContainerStyle={styles.contentContainer}
            showsVerticalScrollIndicator={true}
            bounces={true}  // iOS bounce effect
        >
            <Text>Item 1</Text>
            <Text>Item 2</Text>
            {/* More items */}
        </ScrollView>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    contentContainer: {
        padding: 16,
    },
});
```

### FlatList (Like RecyclerView)

```typescript
import { FlatList, View, Text, StyleSheet } from 'react-native';

interface Item {
    id: string;
    title: string;
    description: string;
}

const MyList: React.FC<{ data: Item[] }> = ({ data }) => {
    const renderItem = ({ item, index }: { item: Item; index: number }) => (
        <View style={styles.item}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.description}>{item.description}</Text>
        </View>
    );

    const renderSeparator = () => <View style={styles.separator} />;

    const renderEmpty = () => (
        <View style={styles.empty}>
            <Text>No items found</Text>
        </View>
    );

    return (
        <FlatList
            data={data}
            renderItem={renderItem}
            keyExtractor={(item) => item.id}
            ItemSeparatorComponent={renderSeparator}
            ListEmptyComponent={renderEmpty}
            onEndReached={() => {
                // Load more items (pagination)
            }}
            onEndReachedThreshold={0.5}  // Trigger at 50% from bottom
            refreshing={false}
            onRefresh={() => {
                // Pull to refresh
            }}
        />
    );
};

const styles = StyleSheet.create({
    item: {
        padding: 16,
        backgroundColor: '#ffffff',
    },
    title: {
        fontSize: 18,
        fontWeight: 'bold',
    },
    description: {
        fontSize: 14,
        color: '#666666',
        marginTop: 4,
    },
    separator: {
        height: 1,
        backgroundColor: '#eeeeee',
    },
    empty: {
        padding: 32,
        alignItems: 'center',
    },
});
```

### SectionList (Grouped Lists)

```typescript
import { SectionList, View, Text, StyleSheet } from 'react-native';

interface Item {
    id: string;
    name: string;
}

interface Section {
    title: string;
    data: Item[];
}

const MySectionList: React.FC<{ sections: Section[] }> = ({ sections }) => {
    return (
        <SectionList
            sections={sections}
            renderItem={({ item }) => (
                <View style={styles.item}>
                    <Text>{item.name}</Text>
                </View>
            )}
            renderSectionHeader={({ section }) => (
                <View style={styles.header}>
                    <Text style={styles.headerText}>{section.title}</Text>
                </View>
            )}
            keyExtractor={(item) => item.id}
        />
    );
};

const styles = StyleSheet.create({
    header: {
        backgroundColor: '#f0f0f0',
        padding: 12,
    },
    headerText: {
        fontSize: 16,
        fontWeight: 'bold',
    },
    item: {
        padding: 16,
    },
});
```

## Layout System (Flexbox)

React Native uses Flexbox for layout, which is different from Android's constraint/linear layouts.

### Flexbox Basics

```typescript
import { View, StyleSheet } from 'react-native';

const FlexboxExample = () => {
    return (
        // Main container
        <View style={styles.container}>
            {/* Row layout (horizontal) */}
            <View style={styles.row}>
                <View style={styles.box} />
                <View style={styles.box} />
                <View style={styles.box} />
            </View>

            {/* Column layout (vertical) - default */}
            <View style={styles.column}>
                <View style={styles.box} />
                <View style={styles.box} />
                <View style={styles.box} />
            </View>

            {/* Space distribution */}
            <View style={styles.spaceBetween}>
                <View style={styles.box} />
                <View style={styles.box} />
                <View style={styles.box} />
            </View>
        </View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    row: {
        flexDirection: 'row',        // Horizontal
        justifyContent: 'flex-start', // Main axis alignment
        alignItems: 'center',         // Cross axis alignment
    },
    column: {
        flexDirection: 'column',      // Vertical (default)
        justifyContent: 'space-around',
        alignItems: 'stretch',
    },
    spaceBetween: {
        flexDirection: 'row',
        justifyContent: 'space-between',  // Equal space between items
        alignItems: 'center',
    },
    box: {
        width: 50,
        height: 50,
        backgroundColor: '#007AFF',
        margin: 8,
    },
});
```

### Android Layout vs Flexbox

| Android | React Native Flexbox |
|---------|---------------------|
| `android:orientation="horizontal"` | `flexDirection: 'row'` |
| `android:orientation="vertical"` | `flexDirection: 'column'` |
| `android:layout_weight` | `flex: 1` |
| `android:gravity="center"` | `justifyContent: 'center'`, `alignItems: 'center'` |
| `android:layout_gravity` | `alignSelf` |
| `match_parent` | `flex: 1` |
| `wrap_content` | Default behavior |

### Responsive Layouts

```typescript
import { View, StyleSheet, Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');

const ResponsiveLayout = () => {
    return (
        <View style={styles.container}>
            <View style={styles.responsiveBox} />
        </View>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
    },
    responsiveBox: {
        width: width * 0.8,      // 80% of screen width
        height: height * 0.3,     // 30% of screen height
        backgroundColor: '#007AFF',
    },
});
```

## Styling

### StyleSheet vs Android XML

**Android XML:**
```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello"
    android:textSize="20sp"
    android:textColor="#000000"
    android:padding="16dp"
    android:background="@drawable/rounded_bg" />
```

**React Native StyleSheet:**
```typescript
import { Text, StyleSheet } from 'react-native';

const MyText = () => <Text style={styles.text}>Hello</Text>;

const styles = StyleSheet.create({
    text: {
        fontSize: 20,
        color: '#000000',
        padding: 16,
        backgroundColor: '#f0f0f0',
        borderRadius: 8,
    },
});
```

### Style Composition

```typescript
const styles = StyleSheet.create({
    base: {
        padding: 16,
        backgroundColor: '#ffffff',
    },
    primary: {
        backgroundColor: '#007AFF',
        color: '#ffffff',
    },
    large: {
        fontSize: 20,
    },
});

// Combine styles
<View style={[styles.base, styles.primary]} />

// Conditional styles
<View style={[
    styles.base,
    isPrimary && styles.primary,
    isLarge && styles.large
]} />
```

### Dynamic Styles

```typescript
const MyComponent = ({ color, size }: { color: string; size: number }) => {
    const dynamicStyles = StyleSheet.create({
        box: {
            width: size,
            height: size,
            backgroundColor: color,
        },
    });

    return <View style={[styles.base, dynamicStyles.box]} />;
};
```

## Platform-Specific Code

### Platform Module

```typescript
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
    container: {
        padding: Platform.OS === 'ios' ? 20 : 16,
        ...Platform.select({
            ios: {
                shadowColor: '#000',
                shadowOffset: { width: 0, height: 2 },
                shadowOpacity: 0.25,
                shadowRadius: 3.84,
            },
            android: {
                elevation: 5,
            },
        }),
    },
});

// Platform check
if (Platform.OS === 'android') {
    // Android-specific code
}

// Platform version
if (Platform.Version >= 21) {
    // Android API 21+
}
```

### Platform-Specific Files

```
MyComponent.tsx        # Shared code
MyComponent.ios.tsx    # iOS-specific
MyComponent.android.tsx # Android-specific
```

```typescript
// Import automatically picks the right file
import MyComponent from './MyComponent';
```

## Native Modules and Bridges

### Using Native Modules

```typescript
import { NativeModules } from 'react-native';

const { MyNativeModule } = NativeModules;

// Call native method
MyNativeModule.doSomething(param1, param2)
    .then(result => console.log(result))
    .catch(error => console.error(error));
```

### Popular Native Libraries

#### React Native Gesture Handler

```typescript
import { GestureHandlerRootView, GestureDetector, Gesture } from 'react-native-gesture-handler';

const MyComponent = () => {
    const tap = Gesture.Tap()
        .onStart(() => console.log('Tap started'))
        .onEnd(() => console.log('Tap ended'));

    const pan = Gesture.Pan()
        .onUpdate((e) => {
            console.log('Pan:', e.translationX, e.translationY);
        });

    return (
        <GestureHandlerRootView style={{ flex: 1 }}>
            <GestureDetector gesture={tap}>
                <View />
            </GestureDetector>
        </GestureHandlerRootView>
    );
};
```

#### React Native Reanimated

```typescript
import Animated, {
    useSharedValue,
    useAnimatedStyle,
    withSpring,
} from 'react-native-reanimated';

const MyAnimatedComponent = () => {
    const offset = useSharedValue(0);

    const animatedStyles = useAnimatedStyle(() => ({
        transform: [{ translateX: offset.value }],
    }));

    const moveBox = () => {
        offset.value = withSpring(offset.value + 100);
    };

    return (
        <>
            <Animated.View style={[styles.box, animatedStyles]} />
            <Button title="Move" onPress={moveBox} />
        </>
    );
};
```

## Modal and Overlays

```typescript
import { useState } from 'react';
import { Modal, View, Text, TouchableOpacity, StyleSheet } from 'react-native';

const MyModal = () => {
    const [visible, setVisible] = useState(false);

    return (
        <>
            <TouchableOpacity onPress={() => setVisible(true)}>
                <Text>Open Modal</Text>
            </TouchableOpacity>

            <Modal
                visible={visible}
                animationType="slide"  // 'none', 'slide', 'fade'
                transparent={true}
                onRequestClose={() => setVisible(false)}
            >
                <View style={styles.modalOverlay}>
                    <View style={styles.modalContent}>
                        <Text>Modal Content</Text>
                        <TouchableOpacity onPress={() => setVisible(false)}>
                            <Text>Close</Text>
                        </TouchableOpacity>
                    </View>
                </View>
            </Modal>
        </>
    );
};

const styles = StyleSheet.create({
    modalOverlay: {
        flex: 1,
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
        justifyContent: 'center',
        alignItems: 'center',
    },
    modalContent: {
        backgroundColor: '#ffffff',
        borderRadius: 12,
        padding: 24,
        width: '80%',
    },
});
```

## SafeAreaView (iOS Notch Handling)

```typescript
import { SafeAreaView, StyleSheet } from 'react-native';

const MyScreen = () => {
    return (
        <SafeAreaView style={styles.container}>
            {/* Content automatically avoids notches and home indicator */}
        </SafeAreaView>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#ffffff',
    },
});
```

## KeyboardAvoidingView

```typescript
import { KeyboardAvoidingView, Platform, StyleSheet } from 'react-native';

const MyForm = () => {
    return (
        <KeyboardAvoidingView
            behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
            style={styles.container}
        >
            <TextInput placeholder="Email" />
            <TextInput placeholder="Password" secureTextEntry />
        </KeyboardAvoidingView>
    );
};
```

## Best Practices

1. **Use StyleSheet.create()** for style optimization
2. **Extract reusable components** into separate files
3. **Use FlatList for long lists** instead of ScrollView
4. **Implement virtualization** for performance
5. **Handle loading and error states** in all components
6. **Use Platform-specific code** when necessary
7. **Optimize images** (use appropriate sizes and formats)
8. **Avoid inline styles** (they create new objects on each render)
9. **Use memo for expensive components**
10. **Handle safe areas** properly for iOS

## Common Pitfalls

1. **Not using keys in lists** - causes rendering issues
2. **Forgetting to handle keyboard** - UI gets hidden
3. **Not optimizing FlatList** - performance problems
4. **Inline functions in renderItem** - causes unnecessary re-renders
5. **Not handling screen sizes** - looks bad on tablets
6. **Missing safe area handling** - content hidden on iOS
7. **Heavy computations in render** - UI becomes sluggish

## Next Steps

Now that you understand React Native UI components, let's explore [Popular Libraries & Ecosystem](./RN-04-popular-libraries.md) to enhance your apps.
