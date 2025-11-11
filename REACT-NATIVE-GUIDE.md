# React Native & TypeScript Learning Guide for Kotlin Android Developers

A comprehensive guide to help Kotlin Android developers transition to React Native and TypeScript development.

## 🎯 Overview

This guide is specifically designed for Android developers with Kotlin experience who want to learn React Native and TypeScript. We'll draw parallels between Android/Kotlin concepts and their React Native/TypeScript equivalents to accelerate your learning.

## 📚 Table of Contents

1. [TypeScript for Kotlin Developers](./RN-01-typescript-for-kotlin-developers.md)
   - Syntax comparison
   - Type system
   - Language features
   - Collections and data structures
   - Async programming (Promises vs Coroutines)

2. [Project Structure & Lifecycle](./RN-02-project-structure-lifecycle.md)
   - React Native project anatomy
   - Directory organization
   - Android vs React Native comparison
   - Component lifecycle
   - App lifecycle
   - Configuration files (package.json, metro.config.js, etc.)

3. [React Native UI Components](./RN-03-ui-components.md)
   - Component tree concept
   - Functional vs Class Components
   - Major components (View, Text, ScrollView, FlatList, etc.)
   - Layout system (Flexbox)
   - Styling with StyleSheet
   - Platform-specific code
   - Native modules and bridges

4. [Popular Libraries & Ecosystem](./RN-04-popular-libraries.md)
   - Essential packages
   - React Navigation
   - UI libraries (React Native Paper, NativeBase, etc.)
   - Form handling
   - Animation libraries
   - Platform-specific integrations
   - Community vs Expo ecosystem

5. [Network Layer Implementation](./RN-05-network-layer.md)
   - Fetch API
   - Axios
   - REST API integration
   - GraphQL with Apollo Client
   - Serialization/Deserialization
   - Error handling
   - Interceptors and middleware
   - Offline support and caching

6. [State Management Deep Dive](./RN-06-state-management.md)
   - Local state (useState, useReducer)
   - Context API
   - Redux & Redux Toolkit
   - MobX
   - Zustand
   - Recoil
   - Jotai
   - Detailed comparison matrix
   - When to use each solution
   - Migration strategies

7. [Design Patterns & Architecture](./RN-07-design-patterns-architecture.md)
   - Container/Presentational pattern
   - Custom Hooks pattern
   - Higher-Order Components (HOC)
   - Render Props
   - Compound Components
   - MVVM in React Native
   - Clean Architecture adaptation
   - Dependency Injection
   - Repository pattern
   - Service layer pattern

8. [Advanced Topics & Best Practices](./RN-08-advanced-best-practices.md)
   - Performance optimization
   - Code organization
   - Testing strategies (Jest, React Native Testing Library)
   - Debugging techniques
   - Memory management
   - Code splitting
   - Common pitfalls

## 🚀 Getting Started

### Prerequisites
- Basic understanding of Kotlin and Android development
- Node.js and npm/yarn installed
- Android Studio and/or Xcode (for iOS)
- React Native development environment setup

### Installation

#### Using React Native CLI
```bash
# Install React Native CLI
npm install -g react-native-cli

# Create a new React Native project with TypeScript
npx react-native init MyApp --template react-native-template-typescript

# Navigate to project
cd MyApp

# Run on Android
npx react-native run-android

# Run on iOS (macOS only)
npx react-native run-ios
```

#### Using Expo (Recommended for beginners)
```bash
# Install Expo CLI
npm install -g expo-cli

# Create a new project
expo init MyApp

# Choose "blank (TypeScript)" template

# Navigate to project
cd MyApp

# Start development server
expo start
```

### Development Environment Setup
```bash
# Verify Node.js installation
node --version

# Verify npm installation
npm --version

# Install TypeScript globally (optional)
npm install -g typescript

# Verify TypeScript
tsc --version
```

## 🎓 Learning Path

We recommend following the sections in order, especially if you're new to React Native:

**Week 1**: Sections 1-2 (TypeScript basics, project structure & lifecycle)
**Week 2**: Sections 3-4 (UI components, popular libraries)
**Week 3**: Sections 5-6 (Network layer, state management)
**Week 4**: Sections 7-8 (Design patterns, best practices)

## 📖 How to Use This Guide

Each section contains:
- **Concept Overview**: High-level explanation
- **Kotlin/Android Comparison**: How it relates to what you already know
- **Code Examples**: Practical, working TypeScript examples
- **Best Practices**: Industry-standard approaches
- **Common Pitfalls**: What to avoid
- **Real-world Use Cases**: When and why to use specific patterns

## 🔄 Kotlin Android vs React Native Quick Reference

| Android/Kotlin | React Native/TypeScript |
|----------------|-------------------------|
| Activity/Fragment | Screen Component |
| View | Component (View, Text, etc.) |
| RecyclerView | FlatList/SectionList |
| ViewModel | Custom Hooks + State Management |
| LiveData | State + useEffect |
| Coroutines | async/await (Promises) |
| XML Layouts | JSX with StyleSheet |
| Jetpack Compose | React Components |
| Room Database | AsyncStorage, Realm, WatermelonDB |
| Retrofit | Axios, Fetch API |
| Navigation Component | React Navigation |
| Dagger/Hilt | Context API, DI libraries |

## 🔗 Additional Resources

- [Official React Native Documentation](https://reactnative.dev/docs/getting-started)
- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [React Documentation](https://react.dev/)
- [React Navigation](https://reactnavigation.org/)
- [npm Package Repository](https://www.npmjs.com/)
- [Awesome React Native](https://github.com/jondot/awesome-react-native)

## 🤝 Contributing

Feel free to contribute to this guide by opening issues or pull requests!

## 📄 License

MIT License - Feel free to use this guide for learning and teaching purposes.

---

**Ready to start?** Begin with [Section 1: TypeScript for Kotlin Developers](./RN-01-typescript-for-kotlin-developers.md)
