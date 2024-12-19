# Flutter Foreground Task Package Guide

## 1. Purpose and Use Case

The `flutter_foreground_task` package enables Flutter applications to run tasks in the foreground service on Android and iOS. A foreground service allows your app to perform operations that should continue even when the app is in the background, while showing a persistent notification to inform users that the app is active.

Common use cases include:
- Music players that continue playback when minimized
- Location tracking apps that need continuous GPS updates
- File download managers
- Health tracking apps that monitor sensors
- Voice recording applications

## 2. Installation

Add the package to your `pubspec.yaml`:

```yaml
dependencies:
  flutter_foreground_task: ^6.1.2
```

### Android Configuration

Add the following permissions to `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />

<application>
    <service
        android:name="com.pravera.flutter_foreground_task.service.ForegroundService"
        android:foregroundServiceType="location"
        android:exported="false" />
</application>
```

### iOS Configuration

Add the following to `ios/Runner/Info.plist`:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>processing</string>
</array>
```

## 3. Key Features

- Persistent notification display
- Customizable notification appearance
- Task callback handling
- Auto-restart capability
- Battery optimization exclusion
- Configurable update intervals
- Data sharing between foreground service and UI
- Support for both Android and iOS platforms

## 4. Implementation Details

### Basic Setup

1. Create a callback function for your foreground task:

```dart
@pragma('vm:entry-point')
void startCallback() {
  FlutterForegroundTask.setTaskHandler(MyTaskHandler());
}

class MyTaskHandler extends TaskHandler {
  @override
  Future<void> onStart(DateTime timestamp, SendPort? sendPort) async {
    // Initialize the task
  }

  @override
  Future<void> onEvent(DateTime timestamp, SendPort? sendPort) async {
    // Periodic task execution
  }

  @override
  Future<void> onDestroy(DateTime timestamp, SendPort? sendPort) async {
    // Cleanup
  }
}
```

2. Configure the foreground task:

```dart
Future<void> initForegroundTask() async {
  FlutterForegroundTask.init(
    androidNotificationOptions: AndroidNotificationOptions(
      channelId: 'foreground_service',
      channelName: 'Foreground Service Notification',
      channelDescription: 'This notification appears when the foreground service is running.',
      channelImportance: NotificationChannelImportance.LOW,
      priority: NotificationPriority.LOW,
      iconData: const NotificationIconData(
        resType: ResourceType.mipmap,
        resPrefix: ResourcePrefix.ic,
        name: 'launcher',
      ),
    ),
    iosNotificationOptions: const IOSNotificationOptions(
      showNotification: true,
      playSound: false,
    ),
    foregroundTaskOptions: const ForegroundTaskOptions(
      interval: 5000,
      isOnceEvent: false,
      autoRunOnBoot: true,
      allowWifiLock: true,
    ),
  );
}
```

3. Start the foreground service:

```dart
Future<bool> startForegroundService() async {
  if (!await FlutterForegroundTask.canDrawOverlays) {
    final isGranted = await FlutterForegroundTask.openSystemAlertWindowSettings();
    if (!isGranted) {
      print('SYSTEM_ALERT_WINDOW permission denied!');
      return false;
    }
  }

  bool reqResult = await FlutterForegroundTask.isIgnoringBatteryOptimizations;
  if (!reqResult) {
    reqResult = await FlutterForegroundTask.requestIgnoreBatteryOptimization();
  }

  return await FlutterForegroundTask.startService(
    notificationTitle: 'Foreground Service Running',
    notificationText: 'Tap to return to the app',
    callback: startCallback,
  );
}
```

### Managing Tasks

To stop the service:
```dart
await FlutterForegroundTask.stopService();
```

To update the notification:
```dart
await FlutterForegroundTask.updateService(
  notificationTitle: 'Updated Title',
  notificationText: 'Updated notification text',
);
```

## 5. Advanced Use Cases

### Periodic Location Updates

```dart
class LocationTaskHandler extends TaskHandler {
  Location location = Location();
  
  @override
  Future<void> onStart(DateTime timestamp, SendPort? sendPort) async {
    await location.enableBackgroundMode(enable: true);
  }

  @override
  Future<void> onEvent(DateTime timestamp, SendPort? sendPort) async {
    LocationData? locationData = await location.getLocation();
    sendPort?.send({
      'latitude': locationData.latitude,
      'longitude': locationData.longitude,
    });
  }
}
```

### Data Synchronization with UI

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  ReceivePort? _receivePort;

  void _initReceivePort() {
    _receivePort = FlutterForegroundTask.receivePort;
    _receivePort?.listen((message) {
      if (message is Map) {
        // Handle received data
        print('Received: $message');
      }
    });
  }

  @override
  void initState() {
    super.initState();
    _initReceivePort();
  }
}
```

## 6. Platform-Specific Notes

### Android
- Supports true foreground services with persistent notifications
- Requires explicit permissions in AndroidManifest.xml
- Can request battery optimization exclusion
- Supports various notification customization options

### iOS
- Limited background execution time
- No persistent notifications in the same way as Android
- Must declare background modes in Info.plist
- May be terminated by the system in low-memory conditions

## 7. Best Practices

1. **Resource Management**
   - Minimize processing in the foreground task
   - Use appropriate update intervals
   - Clean up resources in onDestroy

2. **User Experience**
   - Provide clear notification messages
   - Allow easy service termination
   - Handle task failures gracefully

3. **Battery Optimization**
   - Request battery optimization exclusion when necessary
   - Use appropriate update intervals
   - Minimize wake locks

## 8. Limitations and Challenges

- iOS background execution restrictions
- Different behavior across Android versions
- Battery drain concerns
- Memory management in long-running tasks
- Platform-specific implementation differences

## 9. Performance Considerations

- Keep task execution lightweight
- Use appropriate update intervals (5-15 seconds recommended)
- Monitor battery impact
- Implement proper error handling
- Clean up resources when the task is destroyed

## 10. Alternative Packages

- `workmanager` for periodic background tasks
- `background_fetch` for periodic background fetch
- `android_alarm_manager_plus` for precise scheduling on Android
- `background_locator` for location-specific background tasks

Choose `flutter_foreground_task` when you need:
- Continuous operation with user awareness
- Real-time updates
- Cross-platform compatibility
- Persistent notification control

## 11. Support and Resources

- GitHub Repository: [flutter_foreground_task](https://github.com/dev-yakuza/flutter_foreground_task)
- Issue Tracker: Available on GitHub repository
- Documentation: README.md in the repository
- Community Support: Flutter Discord channel and GitHub discussions

The package is actively maintained with regular updates and bug fixes.
