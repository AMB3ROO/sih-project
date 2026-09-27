# Kabadiwala Connect - Complete Firebase & Technical Audit Report

**Project Name:** Kabadiwala Connect  
**Firebase Live Project:** `kabadiwalaconnectdb`  
**Target Platforms:** Android & iOS  
**Architecture:** Layered Feature-Based Flutter Architecture with Repository Pattern  

---

## 1. Executive Summary

Kabadiwala Connect connects **Household Scrap Sellers**, **Informal Kabadiwala Collectors**, and **Authorized Industrial Recyclers** into a traceable, circular scrap marketplace.

This audit evaluates the codebase's readiness for production integration with live **Firebase (`kabadiwalaconnectdb`)**, **Just-In-Time (JIT) OS permissions**, **multilingual localization**, and **offline-first local database persistence**.

---

## 2. Firebase Integration Status

| Firebase Service | Current Status | Required Action for Production |
| :--- | :--- | :--- |
| **Firebase Core** | Configured via `DefaultFirebaseOptions` | Wired in `main.dart` with platform options for `kabadiwalaconnectdb` |
| **Firebase Authentication** | Phone OTP SMS & User Profiles | Phone number verification, 6-digit SMS OTP, role assignment (`customer`, `kabadiwala`, `recycler`, `admin`) in Firestore `users/{uid}` |
| **Cloud Firestore** | Domain Collections & Repositories | Typed collections: `users`, `scrapLots`, `pickupOrders`, `offers`, `prices`, `handoverRecords`, `notifications`, `safetyGuides` |
| **Firebase Cloud Storage** | Image Compression & Uploads | Upload path structure `/users/{uid}/`, `/scrapLots/{lotId}/`, `/handover/{handoverId}/` |
| **Cloud Messaging (FCM)** | Deep-Link Notifications | Push notification handlers for order matches, counter-bids, arrival alerts, and payouts |
| **Firebase Crashlytics** | Configured in Production Setup | Crash reporting & non-fatal exception tracking |
| **Firebase Remote Config** | Dynamic Feature Flags | Config flags for AI scanner enabled/disabled, minimum app version, max image size |
| **Firebase App Check** | Security Attestation | SafetyNet/Play Integrity (Android) & App Attest/DeviceCheck (iOS) attestation |

---

## 3. Package & Dependency Matrix (`pubspec.yaml`)

### Existing Installed Dependencies:
- `provider: ^6.1.2`: Centralized state provider
- `google_fonts: ^6.2.1`: Typography rendering
- `intl: ^0.19.0`: Date, time slot, and currency formatting
- `cupertino_icons: ^1.0.8`: iOS icon set
- `permission_handler: ^11.3.1`: Native Android & iOS OS runtime permissions
- `camera: ^0.11.0`: Live hardware camera viewfinder feed (`CameraController`)
- `image_picker: ^1.1.2`: Gallery photo selection & storage access
- `flutter_svg: ^2.0.10+1`: Vector graphics rendering

### Additional Recommended Dependencies:
- `firebase_core: ^3.12.0`: Core Firebase initialization
- `firebase_auth: ^5.5.1`: Phone OTP SMS authentication
- `cloud_firestore: ^5.6.5`: Firestore database
- `firebase_storage: ^12.4.4`: Cloud storage image uploads
- `shared_preferences: ^2.5.2`: Local persistence for language & camera permission memory

---

## 4. Device OS Permissions Audit

### Android Manifest (`AndroidManifest.xml`):
```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

### iOS Info Dictionary (`Info.plist`):
```xml
<key>NSCameraUsageDescription</key>
<string>Kabadiwala Connect AI Vision needs camera access to capture scrap photos and calculate instant payout quotes.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Kabadiwala Connect AI Vision needs photo library access to upload scrap photos for material purity analysis.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Kabadiwala Connect needs location access to determine scrap pickup addresses and connect you with nearby collectors.</string>
```

### Contextual Permission Policy (PhonePe/OLX Pattern):
- **Zero Startup Permission Spam**: App opens without asking for camera/location on startup.
- **JIT Action Triggers**: Permissions are requested **only when the user taps an action button** (e.g. "Take Photo", "Upload from Gallery", "Use Current Location").
- **Permanent Permission Memory**: Once camera access is granted, the app **directly opens the live camera feed without asking ever again** until app uninstall.

---

## 5. Multilingual Localization Audit

Languages supported:
1. **English (`en`)**: Primary default UI language
2. **हिन्दी (`hi`)**: Hindi translation map for all buttons, labels, dialogs, order statuses, and safety guides
3. **मराठी (`mr`)**: Marathi translation map for low-literacy vernacular accessibility

Language persistence:
- Stored locally in `SharedPreferences`
- Synced to Firestore user document `users/{uid}/preferredLanguage`
