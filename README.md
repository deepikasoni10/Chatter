# CHATTER 2.O

CHATTER 2.O is an Android one-to-one messaging application written in Java. Users can create an account with a profile image, browse other users, exchange messages in real time, see recent conversations, and receive push notifications when a recipient is unavailable.

## Features

- Sign up with name, email, password, confirmation password, and profile image
- Sign in and persist the signed-in session locally
- Browse other registered users and start a conversation
- Send and receive one-to-one text messages in real time
- Display recent conversations and the latest message
- Show the recipient's online/offline availability
- Send Firebase Cloud Messaging notifications for offline recipients
- Open a chat directly from a notification
- Sign out and clear the local session

## Technology stack

- Java and Android SDK
- AndroidX AppCompat, Material Components, ConstraintLayout, and View Binding
- Firebase Firestore for users, messages, conversations, and presence
- Firebase Cloud Messaging for push notifications
- Retrofit 2 with the Scalars converter for the FCM HTTP request
- RecyclerView adapters for users, conversations, and chat messages
- SharedPreferences for local session and profile data

## Project structure

```text
Chatter/
├── app/
│   └── src/main/
│       ├── java/com/example/chatter2o/
│       │   ├── adapters/       # RecyclerView adapters
│       │   ├── firebase/       # FCM message service
│       │   ├── listeners/      # User and conversation callbacks
│       │   ├── models/         # User and ChatMessage models
│       │   ├── network/        # Retrofit client and FCM API
│       │   ├── utilities/      # Constants and preference storage
│       │   ├── ChatActivity.java
│       │   ├── MainActivity.java
│       │   ├── Signin.java
│       │   ├── Signup.java
│       │   └── UsersActivity.java
│       └── res/                # Layouts, drawables, themes, and strings
├── build.gradle
├── settings.gradle
└── gradlew / gradlew.bat
```

## How the app works

1. `Signin` is the launcher activity. Existing sessions are redirected to `MainActivity`.
2. `Signup` stores a new user document in Firestore and saves the profile in `PreferenceManager`.
3. `MainActivity` loads the profile, refreshes the FCM token, listens for conversation changes, and provides new-chat and sign-out actions.
4. `UsersActivity` reads the `users` collection and excludes the current user.
5. `ChatActivity` listens to both directions of a user pair in the `chat` collection, writes new messages, and maintains a corresponding conversation summary.
6. `BaseActivity` updates the current user's `availability` field when activities pause and resume.
7. `MessagingService` builds a notification from an FCM data payload and opens the related chat when tapped.

## Firestore data model

The app expects these collections and fields:

### `users`

`name`, `email`, `password`, `image` (Base64 profile image), `fcmToken`, and `availability` (`0` or `1`). The Firestore document ID is used as the user ID.

### `chat`

`senderId`, `receiverId`, `message`, and `timestamp`.

### `conversations`

`senderId`, `senderName`, `senderImage`, `receiverId`, `receiverName`, `receiverImage`, `lastMessage`, and `timestamp`.

## Requirements

- Android Studio with an Android SDK that can build compile/target SDK 33
- Java 8-compatible toolchain
- Android device or emulator running API 21 or newer
- A Firebase project with Firestore and Cloud Messaging enabled

## Firebase setup

1. Create or select a Firebase project.
2. Register the Android application with package name `com.example.chatter2o`.
3. Download `google-services.json` and place it at `app/google-services.json`.
4. Create the Firestore database and configure rules that protect user data and messages.
5. Enable Firebase Cloud Messaging.
6. Review the notification configuration before running the app. The current client calls the FCM `/send` endpoint through Retrofit.

`google-services.json` is environment-specific. Do not replace a project configuration with another project's file, and avoid committing credentials or server secrets to a public repository.

## Build and run

From the `Chatter` directory:

```bash
./gradlew assembleDebug
./gradlew test
```

On Windows, use `gradlew.bat` instead:

```powershell
.\gradlew.bat assembleDebug
.\gradlew.bat test
```

Install the generated debug APK on an emulator or connected device, then launch `CHATTER 2.O`.

## Tests

The repository currently contains the default local unit-test and instrumented-test templates. Add tests for authentication, Firestore listeners, message ordering, conversation updates, notification payload handling, and sign-out behavior as the application evolves.

## Important security and reliability notes

The current implementation is suitable for learning/prototyping, but it should not be treated as production-ready without security work:

- Passwords are written to and queried from Firestore as plaintext. Use Firebase Authentication and remove password fields from user documents.
- An FCM server authorization key is hard-coded in `Constants.java`. Revoke/rotate that key and move notification sending to a trusted backend or Firebase Cloud Functions; never ship a server key in an APK.
- Firestore rules must be reviewed and tested to prevent users from reading or modifying other users' data.
- Profile images are stored as Base64 strings in Firestore, which increases document size and bandwidth. Firebase Storage is already included and is a better fit for image files.
- FCM token refresh handling should persist the new token from `onNewToken`.
- The app currently creates conversation documents in both sender/receiver directions and should be reviewed for duplicate conversation behavior and listener cleanup.
- Notification `PendingIntent` flags and notification permission handling should be revisited for newer Android versions.

## License

No license file is currently included. Add a license before distributing the project outside its intended development context.
##Thank you
