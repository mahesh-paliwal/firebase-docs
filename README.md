# firebase push notifications

Firebase Cloud Messaging (FCM) ka use karke aap apni app mein push notifications bhej sakte ho. Yeh guide aapko step-by-step setup karna sikhayegi.

## ✅ Prerequisites

Shuru karne se pehle yeh cheezein ready honi chahiye:

* Firebase account — [console.firebase.google.com](https://console.firebase.google.com/)
* Android Studio ya Xcode (platform ke hisaab se)
* Flutter SDK ya React Native CLI (agar cross-platform app hai)
* Internet connection

{% hint style="info" %}
FCM bilkul **free** hai — koi limit nahi hai notifications bhejne ki.
{% endhint %}

## 🔥 Firebase Project Setup

{% stepper %}
{% step %}
#### Step 1 — Naya Project Banao

1. [Firebase Console](https://console.firebase.google.com/) kholein
2. **"Add Project"** button par click karein
3. Project ka naam dein (jaise: `MyApp`)
4. Google Analytics enable/disable karein → **Continue**
5. **"Create Project"** par click karein
{% endstep %}

{% step %}
#### Step 2 — App Register Karein

Firebase console mein jaao aur apni platform choose karein:

| Platform | Icon          |
| -------- | ------------- |
| Android  | `</> Android` |
| iOS      | `</> Apple`   |
| Web      | `</> Web`     |
{% endstep %}
{% endstepper %}

## 🤖 Android Setup

{% stepper %}
{% step %}
#### Step 1 — App Register Karein

1. Firebase Console → **Project Settings** → **Add App** → Android icon
2. **Package name** daalein (jaise: `com.example.myapp`)
3. **"Register App"** par click karein
4. `google-services.json` file download karein
{% endstep %}

{% step %}
#### Step 2 — google-services.json Place Karein

```
MyApp/
└── android/
    └── app/
        └── google-services.json   ✅ yahan rakho
```
{% endstep %}

{% step %}
#### Step 3 — Gradle Dependencies Add Karein

**Project-level `build.gradle`:**

```groovy
buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.4.0'
    }
}
```

**App-level `build.gradle`:**

```groovy
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'  // ← yeh add karein
}

dependencies {
    implementation platform('com.google.firebase:firebase-bom:32.7.0')
    implementation 'com.google.firebase:firebase-messaging'
}
```
{% endstep %}

{% step %}
#### Step 4 — FCM Service Class Banao

```kotlin
// MyFirebaseMessagingService.kt

import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage
import android.util.Log

class MyFirebaseMessagingService : FirebaseMessagingService() {

    // Naya token milne par
    override fun onNewToken(token: String) {
        super.onNewToken(token)
        Log.d("FCM", "New Token: $token")
        // Token ko apne server par save karein
    }

    // Notification aane par
    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        super.onMessageReceived(remoteMessage)

        remoteMessage.notification?.let {
            Log.d("FCM", "Title: ${it.title}, Body: ${it.body}")
            showNotification(it.title ?: "", it.body ?: "")
        }
    }

    private fun showNotification(title: String, body: String) {
        // Yahan apni notification dikhane ki logic likhein
    }
}
```

**AndroidManifest.xml mein service register karein:**

```xml
<service
    android:name=".MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```
{% endstep %}
{% endstepper %}

## 🍎 iOS Setup

{% stepper %}
{% step %}
#### Step 1 — App Register Karein

1. Firebase Console → **Add App** → Apple icon
2. **Bundle ID** daalein (jaise: `com.example.myapp`)
3. `GoogleService-Info.plist` download karein
4. Ise Xcode project mein **drag & drop** karein
{% endstep %}

{% step %}
#### Step 2 — APNs Certificate Setup

1. [Apple Developer Portal](https://developer.apple.com/) kholo
2. **Certificates → Keys** → Naya key create karein
3. **Apple Push Notifications service (APNs)** enable karein
4. `.p8` file download karein
5. Firebase Console → Project Settings → **Cloud Messaging** → iOS App → APNs key upload karein
{% endstep %}

{% step %}
#### Step 3 — Xcode Capabilities

1. Xcode → Target → **Signing & Capabilities**
2. **+ Capability** → **Push Notifications** add karein
3. **Background Modes** → **Remote notifications** tick karein
{% endstep %}

{% step %}
#### Step 4 — AppDelegate Code

```swift
// AppDelegate.swift

import UIKit
import Firebase
import UserNotifications

@main
class AppDelegate: UIResponder, UIApplicationDelegate,
                   UNUserNotificationCenterDelegate,
                   MessagingDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

        FirebaseApp.configure()

        // Permission maango
        UNUserNotificationCenter.current().delegate = self
        let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
        UNUserNotificationCenter.current().requestAuthorization(options: authOptions) { _, _ in }
        application.registerForRemoteNotifications()

        Messaging.messaging().delegate = self
        return true
    }

    // Token milne par
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        print("FCM Token: \(fcmToken ?? "")")
        // Token ko server par save karein
    }
}
```
{% endstep %}
{% endstepper %}

## 💙 Flutter / React Native Setup

{% tabs %}
{% tab title="Flutter" %}
**Package install karein:**

```bash
flutter pub add firebase_messaging
```

**`main.dart` mein:**

```dart
import 'package:firebase_messaging/firebase_messaging.dart';

// Background message handler (top-level function hona chahiye)
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  print("Background message: ${message.notification?.title}");
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();

  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);

  // Permission maango (iOS ke liye zaroori)
  await FirebaseMessaging.instance.requestPermission();

  // Token lo
  String? token = await FirebaseMessaging.instance.getToken();
  print("FCM Token: $token");

  runApp(MyApp());
}
```
{% endtab %}

{% tab title="React Native" %}
**Package install karein:**

```bash
npm install @react-native-firebase/app @react-native-firebase/messaging
```

**Usage:**

```javascript
import messaging from '@react-native-firebase/messaging';

async function setupFCM() {
  // Permission maango
  const authStatus = await messaging().requestPermission();
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL;

  if (enabled) {
    // Token lo
    const token = await messaging().getToken();
    console.log('FCM Token:', token);

    // Foreground notification listener
    messaging().onMessage(async remoteMessage => {
      console.log('Notification:', remoteMessage.notification);
    });
  }
}
```
{% endtab %}
{% endtabs %}

## 📨 Notification Bhejne ka Tarika

{% stepper %}
{% step %}
#### Step 1 — Firebase Console se (Manual)

1. Firebase Console → **Messaging** → **New Campaign**
2. **Notification title** aur **body** likhein
3. Target choose karein (all users / specific topic / device token)
4. **"Send"** par click karein
{% endstep %}

{% step %}
#### Step 2 — Server se (REST API)

```bash
curl -X POST https://fcm.googleapis.com/v1/projects/YOUR_PROJECT_ID/messages:send \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "token": "DEVICE_FCM_TOKEN",
      "notification": {
        "title": "Namaskar! 🙏",
        "body": "Aapki order deliver ho gayi hai."
      },
      "data": {
        "order_id": "12345",
        "screen": "orders"
      }
    }
  }'
```
{% endstep %}

{% step %}
#### Step 3 — Node.js Server se

```javascript
const admin = require('firebase-admin');

// Service account se initialize karein
admin.initializeApp({
  credential: admin.credential.cert(require('./serviceAccountKey.json'))
});

async function sendNotification(deviceToken, title, body) {
  const message = {
    token: deviceToken,
    notification: { title, body },
    data: { click_action: 'FLUTTER_NOTIFICATION_CLICK' }
  };

  try {
    const response = await admin.messaging().send(message);
    console.log('Notification bheji gayi:', response);
  } catch (error) {
    console.error('Error:', error);
  }
}

// Use karein
sendNotification('DEVICE_TOKEN', '📦 Order Update', 'Aapki delivery aa rahi hai!');
```
{% endstep %}
{% endstepper %}

## 📝 Code Examples

### Topic-based Notification

Ek topic subscribe karein aur saare subscribed users ko ek saath notification bhejen:

```kotlin
// Android — Topic subscribe karein
FirebaseMessaging.getInstance().subscribeToTopic("offers")
    .addOnCompleteListener { task ->
        if (task.isSuccessful) Log.d("FCM", "Topic subscribe hua")
    }
```

```javascript
// Node.js — Topic par notification bhejein
const message = {
  topic: 'offers',
  notification: {
    title: '🎉 Special Offer!',
    body: 'Aaj 50% discount hai!'
  }
};
await admin.messaging().send(message);
```

### Data-only Notification (Silent Push)

```javascript
const message = {
  token: deviceToken,
  data: {
    type: 'SYNC',
    timestamp: Date.now().toString()
  }
  // notification field nahi hai — silent push
};
```

## 🔧 Troubleshooting

### Token nahi mil raha

* `google-services.json` sahi jagah par hai?
*   Internet permission `AndroidManifest.xml` mein add ki?

    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    ```

### iOS par notification nahi aa rahi

* APNs key Firebase mein upload ki?
* Xcode mein Push Notification capability add ki?
* Real device use kar rahe ho? (Simulator par FCM kaam nahi karta)

### Notification background mein nahi aati (Android)

* `onBackgroundMessage` handler top-level function hai?
* App ko battery optimization se exempt karo

### `google-services.json` outdated hai

Firebase Console → Project Settings → Google Services file dobara download karein

## 📚 Helpful Links

* [Firebase Official Docs](https://firebase.google.com/docs/cloud-messaging)
* [FCM REST API Reference](https://firebase.google.com/docs/reference/fcm/rest)
* [FlutterFire Documentation](https://firebase.flutter.dev/docs/messaging/overview)
* [React Native Firebase](https://rnfirebase.io/messaging/usage)

{% hint style="info" %}
FCM token har baar app reinstall hone par change hota hai. Isliye token ko hamesha server par update karna mat bhoolein.
{% endhint %}
