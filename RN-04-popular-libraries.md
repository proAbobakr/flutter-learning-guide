# Popular Libraries & Ecosystem

## Essential Libraries Comparison

### Android vs React Native Libraries

| Android | React Native | Purpose |
|---------|--------------|---------|
| Retrofit | Axios, Fetch API | HTTP client |
| Room | AsyncStorage, Realm, WatermelonDB | Local database |
| Glide/Picasso | react-native-fast-image | Image loading |
| Navigation Component | React Navigation | Navigation |
| Dagger/Hilt | Context API, InversifyJS | Dependency injection |
| WorkManager | react-native-background-task | Background tasks |
| Firebase | @react-native-firebase/* | Backend services |
| Crashlytics | @sentry/react-native | Crash reporting |
| LeakCanary | react-devtools | Memory profiling |

## Navigation

### React Navigation (Most Popular)

Installation:
```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
npm install @react-navigation/native-stack @react-navigation/bottom-tabs
```

#### Stack Navigation (Like Fragment Stack)

```typescript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// Define navigation params
type RootStackParamList = {
    Home: undefined;
    Details: { itemId: string; title: string };
    Profile: { userId: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

const App = () => {
    return (
        <NavigationContainer>
            <Stack.Navigator
                initialRouteName="Home"
                screenOptions={{
                    headerStyle: { backgroundColor: '#007AFF' },
                    headerTintColor: '#fff',
                }}
            >
                <Stack.Screen
                    name="Home"
                    component={HomeScreen}
                    options={{ title: 'My Home' }}
                />
                <Stack.Screen
                    name="Details"
                    component={DetailsScreen}
                />
                <Stack.Screen
                    name="Profile"
                    component={ProfileScreen}
                    options={{ headerShown: false }}
                />
            </Stack.Navigator>
        </NavigationContainer>
    );
};
```

#### Using Navigation in Components

```typescript
import { useNavigation, useRoute, RouteProp } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';

type HomeScreenNavigationProp = NativeStackNavigationProp<
    RootStackParamList,
    'Home'
>;

type DetailsScreenRouteProp = RouteProp<RootStackParamList, 'Details'>;

const HomeScreen = () => {
    const navigation = useNavigation<HomeScreenNavigationProp>();

    const goToDetails = () => {
        navigation.navigate('Details', {
            itemId: '123',
            title: 'Item Title',
        });
    };

    return (
        <TouchableOpacity onPress={goToDetails}>
            <Text>Go to Details</Text>
        </TouchableOpacity>
    );
};

const DetailsScreen = () => {
    const route = useRoute<DetailsScreenRouteProp>();
    const navigation = useNavigation();
    const { itemId, title } = route.params;

    return (
        <View>
            <Text>{title}</Text>
            <Text>Item ID: {itemId}</Text>
            <Button
                title="Go Back"
                onPress={() => navigation.goBack()}
            />
        </View>
    );
};
```

#### Bottom Tab Navigation (Like BottomNavigationView)

```typescript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import Icon from 'react-native-vector-icons/Ionicons';

const Tab = createBottomTabNavigator();

const TabNavigator = () => {
    return (
        <Tab.Navigator
            screenOptions={({ route }) => ({
                tabBarIcon: ({ focused, color, size }) => {
                    let iconName: string;

                    if (route.name === 'Home') {
                        iconName = focused ? 'home' : 'home-outline';
                    } else if (route.name === 'Profile') {
                        iconName = focused ? 'person' : 'person-outline';
                    }

                    return <Icon name={iconName} size={size} color={color} />;
                },
                tabBarActiveTintColor: '#007AFF',
                tabBarInactiveTintColor: 'gray',
            })}
        >
            <Tab.Screen name="Home" component={HomeScreen} />
            <Tab.Screen name="Search" component={SearchScreen} />
            <Tab.Screen name="Profile" component={ProfileScreen} />
        </Tab.Navigator>
    );
};
```

#### Drawer Navigation (Like NavigationDrawer)

```typescript
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

const DrawerNavigator = () => {
    return (
        <Drawer.Navigator
            screenOptions={{
                drawerStyle: {
                    backgroundColor: '#f0f0f0',
                    width: 280,
                },
            }}
        >
            <Drawer.Screen name="Home" component={HomeScreen} />
            <Drawer.Screen name="Settings" component={SettingsScreen} />
        </Drawer.Navigator>
    );
};
```

## UI Component Libraries

### React Native Paper (Material Design)

```bash
npm install react-native-paper
npm install react-native-vector-icons
```

```typescript
import { Provider as PaperProvider, Button, Card, Title } from 'react-native-paper';

const App = () => {
    return (
        <PaperProvider>
            <Card>
                <Card.Content>
                    <Title>Card Title</Title>
                </Card.Content>
                <Card.Actions>
                    <Button mode="contained">Save</Button>
                    <Button mode="outlined">Cancel</Button>
                </Card.Actions>
            </Card>
        </PaperProvider>
    );
};
```

### React Native Elements

```bash
npm install @rneui/themed @rneui/base
```

```typescript
import { Button, Card, Input } from '@rneui/themed';

const MyComponent = () => {
    return (
        <>
            <Input
                placeholder="Enter name"
                leftIcon={{ type: 'font-awesome', name: 'user' }}
            />
            <Button
                title="Submit"
                buttonStyle={{ backgroundColor: '#007AFF' }}
                loading={false}
            />
            <Card>
                <Card.Title>Card Title</Card.Title>
                <Card.Divider />
                <Text>Card Content</Text>
            </Card>
        </>
    );
};
```

### NativeBase

```bash
npm install native-base
```

```typescript
import { NativeBaseProvider, Box, Button, VStack } from 'native-base';

const App = () => {
    return (
        <NativeBaseProvider>
            <VStack space={4} alignItems="center">
                <Box bg="primary.500" p={4} rounded="md">
                    <Text color="white">Box Component</Text>
                </Box>
                <Button colorScheme="primary">Press Me</Button>
            </VStack>
        </NativeBaseProvider>
    );
};
```

## Form Handling

### React Hook Form

```bash
npm install react-hook-form
```

```typescript
import { useForm, Controller } from 'react-hook-form';
import { TextInput, Button, View } from 'react-native';

interface FormData {
    email: string;
    password: string;
}

const LoginForm = () => {
    const {
        control,
        handleSubmit,
        formState: { errors },
    } = useForm<FormData>({
        defaultValues: {
            email: '',
            password: '',
        },
    });

    const onSubmit = (data: FormData) => {
        console.log('Form data:', data);
    };

    return (
        <View>
            <Controller
                control={control}
                rules={{
                    required: 'Email is required',
                    pattern: {
                        value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
                        message: 'Invalid email address',
                    },
                }}
                render={({ field: { onChange, onBlur, value } }) => (
                    <TextInput
                        onBlur={onBlur}
                        onChangeText={onChange}
                        value={value}
                        placeholder="Email"
                        keyboardType="email-address"
                    />
                )}
                name="email"
            />
            {errors.email && <Text>{errors.email.message}</Text>}

            <Controller
                control={control}
                rules={{ required: 'Password is required' }}
                render={({ field: { onChange, onBlur, value } }) => (
                    <TextInput
                        onBlur={onBlur}
                        onChangeText={onChange}
                        value={value}
                        placeholder="Password"
                        secureTextEntry
                    />
                )}
                name="password"
            />
            {errors.password && <Text>{errors.password.message}</Text>}

            <Button title="Submit" onPress={handleSubmit(onSubmit)} />
        </View>
    );
};
```

### Formik (Alternative)

```bash
npm install formik yup
```

```typescript
import { Formik } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object().shape({
    email: Yup.string()
        .email('Invalid email')
        .required('Email is required'),
    password: Yup.string()
        .min(6, 'Password must be at least 6 characters')
        .required('Password is required'),
});

const LoginForm = () => {
    return (
        <Formik
            initialValues={{ email: '', password: '' }}
            validationSchema={validationSchema}
            onSubmit={(values) => {
                console.log('Form data:', values);
            }}
        >
            {({ handleChange, handleBlur, handleSubmit, values, errors, touched }) => (
                <View>
                    <TextInput
                        onChangeText={handleChange('email')}
                        onBlur={handleBlur('email')}
                        value={values.email}
                        placeholder="Email"
                    />
                    {touched.email && errors.email && (
                        <Text>{errors.email}</Text>
                    )}

                    <TextInput
                        onChangeText={handleChange('password')}
                        onBlur={handleBlur('password')}
                        value={values.password}
                        placeholder="Password"
                        secureTextEntry
                    />
                    {touched.password && errors.password && (
                        <Text>{errors.password}</Text>
                    )}

                    <Button title="Submit" onPress={handleSubmit} />
                </View>
            )}
        </Formik>
    );
};
```

## Animation Libraries

### React Native Reanimated (Recommended)

```bash
npm install react-native-reanimated
```

```typescript
import Animated, {
    useSharedValue,
    useAnimatedStyle,
    withTiming,
    withSpring,
    withRepeat,
} from 'react-native-reanimated';

const AnimatedBox = () => {
    const offset = useSharedValue(0);
    const opacity = useSharedValue(1);

    const animatedStyles = useAnimatedStyle(() => ({
        transform: [
            { translateX: offset.value },
            { scale: 1 + offset.value / 200 },
        ],
        opacity: opacity.value,
    }));

    const animateBox = () => {
        // Spring animation
        offset.value = withSpring(100);

        // Timing animation
        opacity.value = withTiming(0.5, { duration: 300 });

        // Repeat animation
        offset.value = withRepeat(
            withTiming(100, { duration: 1000 }),
            -1,  // Infinite
            true // Reverse
        );
    };

    return (
        <>
            <Animated.View style={[styles.box, animatedStyles]} />
            <Button title="Animate" onPress={animateBox} />
        </>
    );
};
```

### Lottie (JSON Animations)

```bash
npm install lottie-react-native
```

```typescript
import LottieView from 'lottie-react-native';

const LottieAnimation = () => {
    return (
        <LottieView
            source={require('./animation.json')}
            autoPlay
            loop
            style={{ width: 200, height: 200 }}
        />
    );
};
```

## Storage Solutions

### AsyncStorage (Like SharedPreferences)

```bash
npm install @react-native-async-storage/async-storage
```

```typescript
import AsyncStorage from '@react-native-async-storage/async-storage';

// Save data
const saveData = async (key: string, value: string) => {
    try {
        await AsyncStorage.setItem(key, value);
    } catch (error) {
        console.error('Error saving data:', error);
    }
};

// Save object
const saveObject = async (key: string, value: object) => {
    try {
        await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
        console.error('Error saving object:', error);
    }
};

// Get data
const getData = async (key: string): Promise<string | null> => {
    try {
        return await AsyncStorage.getItem(key);
    } catch (error) {
        console.error('Error getting data:', error);
        return null;
    }
};

// Get object
const getObject = async <T,>(key: string): Promise<T | null> => {
    try {
        const jsonValue = await AsyncStorage.getItem(key);
        return jsonValue != null ? JSON.parse(jsonValue) : null;
    } catch (error) {
        console.error('Error getting object:', error);
        return null;
    }
};

// Remove data
const removeData = async (key: string) => {
    try {
        await AsyncStorage.removeItem(key);
    } catch (error) {
        console.error('Error removing data:', error);
    }
};

// Clear all
const clearAll = async () => {
    try {
        await AsyncStorage.clear();
    } catch (error) {
        console.error('Error clearing storage:', error);
    }
};
```

### Realm (Like Room Database)

```bash
npm install realm
```

```typescript
import Realm from 'realm';

// Define schema
class User extends Realm.Object {
    static schema = {
        name: 'User',
        properties: {
            id: 'int',
            name: 'string',
            email: 'string?',
            createdAt: 'date',
        },
        primaryKey: 'id',
    };
}

// Open database
const realm = await Realm.open({
    schema: [User],
    schemaVersion: 1,
});

// Create
realm.write(() => {
    realm.create('User', {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
        createdAt: new Date(),
    });
});

// Read
const users = realm.objects('User');
const user = realm.objectForPrimaryKey('User', 1);

// Update
realm.write(() => {
    user.name = 'Jane Doe';
});

// Delete
realm.write(() => {
    realm.delete(user);
});

// Query
const filteredUsers = realm
    .objects('User')
    .filtered('name CONTAINS[c] $0', 'John');
```

### WatermelonDB (High-Performance)

```bash
npm install @nozbe/watermelondb
```

```typescript
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { Model } from '@nozbe/watermelondb';
import { field, date } from '@nozbe/watermelondb/decorators';

// Model definition
class User extends Model {
    static table = 'users';

    @field('name') name!: string;
    @field('email') email!: string;
    @date('created_at') createdAt!: Date;
}

// Database setup
const adapter = new SQLiteAdapter({
    schema: mySchema,
    migrations: myMigrations,
});

const database = new Database({
    adapter,
    modelClasses: [User],
});

// Usage
const users = await database.get('users').query().fetch();
```

## Date/Time Handling

### date-fns (Recommended)

```bash
npm install date-fns
```

```typescript
import { format, addDays, differenceInDays, parseISO } from 'date-fns';

const now = new Date();
const formatted = format(now, 'yyyy-MM-dd HH:mm:ss');
const tomorrow = addDays(now, 1);
const daysDiff = differenceInDays(new Date(2024, 11, 31), now);
const parsed = parseISO('2024-01-15T10:30:00');
```

### Moment.js (Popular but heavy)

```bash
npm install moment
```

```typescript
import moment from 'moment';

const now = moment();
const formatted = now.format('YYYY-MM-DD HH:mm:ss');
const tomorrow = moment().add(1, 'days');
const daysDiff = moment('2024-12-31').diff(moment(), 'days');
```

## Image Handling

### react-native-fast-image

```bash
npm install react-native-fast-image
```

```typescript
import FastImage from 'react-native-fast-image';

const MyImage = () => {
    return (
        <FastImage
            style={{ width: 200, height: 200 }}
            source={{
                uri: 'https://example.com/image.jpg',
                priority: FastImage.priority.high,
                cache: FastImage.cacheControl.immutable,
            }}
            resizeMode={FastImage.resizeMode.cover}
        />
    );
};
```

### react-native-image-picker

```bash
npm install react-native-image-picker
```

```typescript
import { launchImageLibrary, launchCamera } from 'react-native-image-picker';

const pickImage = () => {
    launchImageLibrary({
        mediaType: 'photo',
        quality: 0.8,
    }, (response) => {
        if (response.assets) {
            const image = response.assets[0];
            console.log('Image URI:', image.uri);
        }
    });
};

const takePhoto = () => {
    launchCamera({
        mediaType: 'photo',
        quality: 0.8,
    }, (response) => {
        if (response.assets) {
            const photo = response.assets[0];
            console.log('Photo URI:', photo.uri);
        }
    });
};
```

## Permissions

### react-native-permissions

```bash
npm install react-native-permissions
```

```typescript
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

const checkCameraPermission = async () => {
    const result = await check(PERMISSIONS.ANDROID.CAMERA);

    switch (result) {
        case RESULTS.UNAVAILABLE:
            console.log('Feature not available');
            break;
        case RESULTS.DENIED:
            console.log('Permission denied');
            const requestResult = await request(PERMISSIONS.ANDROID.CAMERA);
            break;
        case RESULTS.GRANTED:
            console.log('Permission granted');
            break;
        case RESULTS.BLOCKED:
            console.log('Permission blocked');
            break;
    }
};
```

## Testing Libraries

### Jest (Built-in)

```typescript
// MyComponent.test.tsx
import { render, fireEvent } from '@testing-library/react-native';
import MyComponent from './MyComponent';

describe('MyComponent', () => {
    it('renders correctly', () => {
        const { getByText } = render(<MyComponent />);
        expect(getByText('Hello')).toBeTruthy();
    });

    it('handles button press', () => {
        const { getByText } = render(<MyComponent />);
        const button = getByText('Press Me');
        fireEvent.press(button);
        // Assert expected behavior
    });
});
```

### React Native Testing Library

```bash
npm install --save-dev @testing-library/react-native
```

```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react-native';

describe('LoginScreen', () => {
    it('submits form with valid data', async () => {
        render(<LoginScreen />);

        const emailInput = screen.getByPlaceholderText('Email');
        const passwordInput = screen.getByPlaceholderText('Password');
        const submitButton = screen.getByText('Submit');

        fireEvent.changeText(emailInput, 'test@example.com');
        fireEvent.changeText(passwordInput, 'password123');
        fireEvent.press(submitButton);

        await waitFor(() => {
            expect(screen.getByText('Success')).toBeTruthy();
        });
    });
});
```

## Utility Libraries

### Lodash

```bash
npm install lodash
npm install --save-dev @types/lodash
```

```typescript
import _ from 'lodash';

const numbers = [1, 2, 3, 4, 5];
const doubled = _.map(numbers, n => n * 2);
const grouped = _.groupBy(users, 'role');
const debounced = _.debounce(searchFunction, 300);
```

## Platform-Specific Integrations

### Expo (Easier Development)

```bash
npm install -g expo-cli
expo init MyApp
```

Expo provides many ready-to-use APIs:
- Camera
- Location
- Notifications
- File System
- Media Library
- Sensors
- And more...

## Best Practices

1. **Choose the right library** for your needs (bundle size matters)
2. **Check maintenance status** (last update, GitHub stars, issues)
3. **Read documentation** thoroughly before integrating
4. **Consider native dependencies** (more complex builds)
5. **Use TypeScript types** for better development experience
6. **Test library compatibility** with your React Native version
7. **Monitor bundle size** with metro-bundler
8. **Prefer pure JavaScript libraries** when possible
9. **Check Expo compatibility** if using Expo

## Common Library Combinations

### Typical Production Setup

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-native": "^0.72.0",
    "@react-navigation/native": "^6.1.0",
    "@react-navigation/native-stack": "^6.9.0",
    "axios": "^1.4.0",
    "@tanstack/react-query": "^4.29.0",
    "zustand": "^4.3.0",
    "react-hook-form": "^7.45.0",
    "date-fns": "^2.30.0",
    "@react-native-async-storage/async-storage": "^1.19.0",
    "react-native-reanimated": "^3.3.0",
    "react-native-gesture-handler": "^2.12.0"
  }
}
```

## Next Steps

Now that you're familiar with the ecosystem, let's dive into [Network Layer Implementation](./RN-05-network-layer.md) to handle API calls effectively.
