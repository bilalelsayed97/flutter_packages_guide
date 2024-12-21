# Hydrated Bloc Package: A Comprehensive Guide

## 1. Purpose and Use Case

The `hydrated_bloc` package is an extension to `flutter_bloc` that automatically persists and restores bloc states. It enables your Flutter application to maintain state even after the app is closed and reopened.

### Key Benefits
- Automatic state persistence
- Seamless integration with flutter_bloc
- Minimal additional code required
- Built-in serialization support
- Fast synchronous storage operations

### Ideal Scenarios
- Apps requiring offline data persistence
- Applications needing state restoration after restarts
- User preference management
- Shopping carts and forms
- Apps with complex state that needs to survive restarts

## 2. Installation

Add the following dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  hydrated_bloc: ^9.1.2
  flutter_bloc: ^8.1.3
  path_provider: ^2.1.1

dev_dependencies:
  build_runner: ^2.4.6
  json_serializable: ^6.7.1
```

Run:
```bash
flutter pub get
```

## 3. Key Features

- **Automatic State Persistence**: Saves bloc state to local storage
- **JSON Serialization**: Built-in support for JSON encoding/decoding
- **Storage Configuration**: Customizable storage directory
- **Migration Support**: Tools for handling state version changes
- **Cross-Platform**: Works on all Flutter platforms
- **Type Safety**: Strong typing for state persistence

## 4. Implementation Details

### Basic Setup
```dart
import 'package:hydrated_bloc/hydrated_bloc.dart';
import 'package:path_provider/path_provider.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  HydratedBloc.storage = await HydratedStorage.build(
    storageDirectory: await getTemporaryDirectory(),
  );
  
  runApp(MyApp());
}
```

### Creating a Hydrated Bloc
```dart
class CounterBloc extends HydratedBloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState(0)) {
    on<IncrementPressed>((event, emit) {
      emit(CounterState(state.count + 1));
    });
  }

  @override
  CounterState? fromJson(Map<String, dynamic> json) {
    return CounterState(json['count'] as int);
  }

  @override
  Map<String, dynamic>? toJson(CounterState state) {
    return {'count': state.count};
  }
}
```

### State Class
```dart
class CounterState {
  final int count;
  
  CounterState(this.count);
  
  // For more complex states, use json_serializable
  factory CounterState.fromJson(Map<String, dynamic> json) => 
    CounterState(json['count'] as int);
  
  Map<String, dynamic> toJson() => {'count': count};
}
```

## 5. Advanced Examples

### Complex State with JSON Serialization
```dart
@JsonSerializable()
class UserState {
  final String name;
  final List<String> preferences;
  final Map<String, dynamic> settings;

  UserState({
    required this.name,
    required this.preferences,
    required this.settings,
  });

  factory UserState.fromJson(Map<String, dynamic> json) => 
    _$UserStateFromJson(json);
  
  Map<String, dynamic> toJson() => _$UserStateToJson(this);
}

class UserBloc extends HydratedBloc<UserEvent, UserState> {
  UserBloc() : super(UserState(
    name: '',
    preferences: [],
    settings: {},
  )) {
    on<UpdateUser>((event, emit) {
      emit(event.newState);
    });
  }

  @override
  UserState? fromJson(Map<String, dynamic> json) =>
    UserState.fromJson(json);

  @override
  Map<String, dynamic>? toJson(UserState state) =>
    state.toJson();
}
```

### State Migration Example
```dart
class SettingsBloc extends HydratedBloc<SettingsEvent, SettingsState> {
  @override
  SettingsState? fromJson(Map<String, dynamic> json) {
    final version = json['version'] as int?;
    
    switch (version) {
      case 1:
        return _migrateV1ToV2(json);
      case 2:
        return SettingsState.fromJson(json);
      default:
        return null;
    }
  }

  SettingsState _migrateV1ToV2(Map<String, dynamic> json) {
    // Migration logic here
    return SettingsState.fromJson({
      ...json,
      'newField': 'default',
      'version': 2,
    });
  }
}
```

## 6. Best Practices

### Do's
- Initialize storage early in app lifecycle
- Implement proper error handling in fromJson/toJson
- Use JSON serializable for complex states
- Keep stored data minimal
- Handle migration scenarios

### Don'ts
- Store sensitive information (use flutter_secure_storage instead)
- Store large amounts of data
- Ignore migration between app versions
- Store redundant or derived data

## 7. Performance Optimization

### Storage Optimization
```dart
class OptimizedBloc extends HydratedBloc<Event, State> {
  @override
  Map<String, dynamic>? toJson(State state) {
    // Only persist essential data
    return {
      'critical_data': state.criticalData,
      // Omit derived or temporary data
    };
  }
  
  @override
  Stream<State> mapEventToState(Event event) async* {
    // Implement caching if needed
    if (_cache.containsKey(event)) {
      yield _cache[event];
      return;
    }
    // Process event
  }
}
```

## 8. Limitations and Considerations

### Known Limitations
- Limited to JSON-serializable data
- Storage size constraints
- Synchronous storage operations
- No built-in encryption
- Platform-specific storage locations

### Storage Size Management
```dart
class StorageManager {
  static Future<void> checkStorageSize() async {
    final directory = await getTemporaryDirectory();
    final files = directory.listSync();
    int totalSize = 0;
    
    for (var file in files) {
      totalSize += await file.length();
    }
    
    if (totalSize > 10 * 1024 * 1024) { // 10MB limit
      // Implement cleanup logic
    }
  }
}
```

## 9. Alternative Solutions

- **SharedPreferences**: Simple key-value storage
- **Hive**: NoSQL database solution
- **SQLite**: Relational database option
- **sembast**: NoSQL alternative
- **get_storage**: GetX storage solution

## 10. Support and Resources

- GitHub Repository: [felangel/hydrated_bloc](https://github.com/felangel/hydrated_bloc)
- Documentation: [hydrated_bloc documentation](https://pub.dev/packages/hydrated_bloc)
- Issue Tracker: GitHub Issues
- Community: Bloc Discord server

## 11. Platform-Specific Notes

### Storage Locations
- **Android**: App-specific directory in internal storage
- **iOS**: App's documents directory
- **Web**: IndexedDB
- **Desktop**: Local app data directory

### Platform Considerations
- Web platform has storage limitations based on browser
- iOS requires proper encryption for sensitive data
- Android may clear cache in low storage situations
