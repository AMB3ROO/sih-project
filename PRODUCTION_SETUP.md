# Kabadiwala Connect - Complete Production Deployment Manual

This manual provides step-by-step instructions for deploying the **Kabadiwala Connect** application to live production backend services.

---

## 1. Firebase Console Steps (`kabadiwalaconnectdb`)

1. **Access Firebase Console**:
   - Open [Firebase Console](https://console.firebase.google.com/) and select project **`kabadiwalaconnectdb`**.

2. **Configure Authentication**:
   - Navigate to **Build → Authentication → Sign-in method**.
   - Enable **Phone** authentication.
   - Under **Settings → Authorized domains**, include `localhost` for local web runs and the exact production web host before testing browser OTP. Web sign-in uses Firebase reCAPTCHA; Android/iOS use native phone verification callbacks.
   - (Optional) Add test phone numbers (e.g. `+91 98765 43210`, OTP `123456`) for App Store / Play Store review testing.

3. **Deploy Firestore Security Rules & Indexes**:
   - Install Firebase CLI: `npm install -g firebase-tools`
   - Login: `firebase login`
   - Deploy Firestore rules & indexes:
     ```bash
     firebase deploy --only firestore:rules,firestore:indexes --project kabadiwalaconnectdb
     ```

4. **Deploy Storage Rules**:
   - Deploy Storage security rules:
     ```bash
     firebase deploy --only storage --project kabadiwalaconnectdb
     ```

---

## 2. Google Cloud Platform (GCP) Steps

1. **Restrict API Keys**:
   - Open [GCP Credentials Console](https://console.cloud.google.com/apis/credentials).
   - Locate the API key for `kabadiwalaconnectdb`.
   - Set **Application Restrictions**:
     - Android: Package name `com.example.kabadiwala_conect` & SHA-1 fingerprint (the current `applicationId` in `android/app/build.gradle.kts`).
     - iOS: Bundle identifier configured in the Xcode project; the Android application ID does not set it.
   - Set **API Restrictions**: Limit usage to *Firebase Services*, *Maps SDK for Android*, and *Maps SDK for iOS*.

---

## 3. Google Play Store Release Commands

1. **Build Release Android App Bundle (AAB)**:
   ```bash
   flutter build appbundle --release
   ```
   Output: `build/app/outputs/bundle/release/app-release.aab`

2. **Upload to Google Play Console**:
   - Go to [Google Play Console](https://play.google.com/console/).
   - Upload `app-release.aab` under Production / Internal Testing track.

---

## 4. Apple App Store Release Commands

1. **Build Release iOS Archive**:
   ```bash
   flutter build ios --release
   ```

2. **Archive & Submit via Xcode**:
   - Open `ios/Runner.xcworkspace` in Xcode.
   - Select **Product → Archive → Distribute App → App Store Connect**.
