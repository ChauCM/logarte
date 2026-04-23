# logarte

In-app debug console and logger for Flutter apps. Inspect network requests, push notifications, navigation, storage operations, and plain logs -- with optional persistent storage that survives app restarts.

> This is a fork of [kamranbekirovyz/logarte](https://github.com/kamranbekirovyz/logarte) with added notification logging and persistent storage.

## Features

- **In-app console** with tabbed views: All, Logging, Network, Database, Navigation, Notifications
- **Persistent storage** -- opt-in JSONL file persistence so logs survive app restarts (useful for background notifications and crash debugging)
- **Network inspector** -- track API calls and responses (Dio interceptor included)
- **Notification inspector** -- log push notification events (received, tapped, subscribed, unsubscribed)
- **Navigation tracking** via `NavigatorObserver`
- **Storage monitor** for local storage operations
- **Password protection** for production access
- **Copy and share** debug logs with your team

## Getting Started

### Install

```yaml
dependencies:
  logarte:
    git:
      url: https://github.com/ChauCM/logarte.git
```

### Initialize

```dart
final Logarte logarte = Logarte(
  password: '1234',
  ignorePassword: kDebugMode,
  onShare: (String content) {
    Share.share(content);
  },
);
```

#### With persistent storage

Logs are in-memory by default. To keep them across app restarts, pass a `FileLogartePersistence`:

```dart
final logarte = Logarte(
  password: '1234',
  persistence: FileLogartePersistence(
    directory: await getApplicationDocumentsDirectory(),
    maxAge: Duration(days: 7),   // auto-prune entries older than 7 days
    maxEntries: 2500,            // keep at most 2500 entries
  ),
);

await logarte.init(); // loads persisted logs from previous sessions
```

### Attach the floating button

```dart
@override
void initState() {
  super.initState();
  logarte.attach(context: context, visible: kDebugMode);
}
```

Or open the console directly:

```dart
logarte.openConsole(context);
```

For production access, use the hidden gesture trigger:

```dart
LogarteMagicalTap(
  logarte: logarte,
  child: Text('App Version 1.0'),
)
```

## Logging

### Network requests

With Dio:

```dart
dio.interceptors.add(LogarteDioInterceptor(logarte));
```

With any HTTP client:

```dart
logarte.network(
  request: NetworkRequestLogarteEntry(
    method: 'POST',
    url: endpoint,
    headers: headers,
    body: body,
  ),
  response: NetworkResponseLogarteEntry(
    statusCode: response.statusCode,
    headers: response.headers,
    body: response.body,
  ),
);
```

### Push notifications

```dart
logarte.notification(
  eventType: NotificationEventType.received,
  messageId: message.messageId,
  title: message.notification?.title,
  body: message.notification?.body,
  topic: message.from,
  data: message.data,
  source: 'foreground',
);
```

Event types: `received`, `tapped`, `subscribed`, `unsubscribed`.

### Navigation

```dart
MaterialApp(
  navigatorObservers: [LogarteNavigatorObserver(logarte)],
)
```

### Storage operations

```dart
logarte.database(
  target: 'language',
  value: 'en',
  source: 'SharedPreferences',
);
```

### Plain messages

```dart
logarte.log('Button clicked');
```

### Custom tab

```dart
final logarte = Logarte(
  customTab: const MyCustomTab(),
);
```

### Disabling specific tabs

```dart
final logarte = Logarte(
  disableNavigationLogs: true,
  disableDatabaseLogs: true,
  disableNotificationLogs: true,
);
```

## Configuration

| Parameter | Type | Default | Description |
|---|---|---|---|
| `password` | `String?` | `null` | Password to protect console access |
| `ignorePassword` | `bool` | `!kReleaseMode` | Skip password in debug mode |
| `persistence` | `LogartePersistence?` | `null` | Opt-in persistent storage |
| `logBufferLength` | `int` | `2500` | Max in-memory entries |
| `disableAllLogs` | `bool` | `false` | Hide the "All" tab |
| `disablePlainLogs` | `bool` | `false` | Hide the "Logging" tab |
| `disableNetworkLogs` | `bool` | `false` | Hide the "Network" tab |
| `disableDatabaseLogs` | `bool` | `false` | Hide the "Database" tab |
| `disableNavigationLogs` | `bool` | `false` | Hide the "Navigation" tab |
| `disableNotificationLogs` | `bool` | `false` | Hide the "Notifications" tab |

## License

MIT License - see [LICENSE](LICENSE).
