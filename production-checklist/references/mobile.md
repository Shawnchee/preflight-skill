# Mobile App Production Checklist (iOS & Android)

---

## 🔴 CRITICAL

### App Store / Play Store Compliance

- [ ] iOS: App built with latest required Xcode version, targeting current minimum iOS SDK (check Apple's current deadline)
- [ ] Android: App targets the current required API level — new apps on Play Store must target latest stable API
- [ ] Android: App distributed as `.aab` (Android App Bundle), NOT legacy APK (required by Play Store)
- [ ] Privacy policy URL is live, accessible, and linked in both App Store Connect and Play Console store listings
- [ ] All requested permissions have user-facing justification strings (iOS: every `NS...UsageDescription` key in Info.plist; Android: runtime permission rationale)
- [ ] Only permissions actually used by the app are requested — remove any leftover/unused permission declarations
- [ ] Demo/test account credentials prepared and entered for App Store reviewers (required if app has login — rejection guaranteed without this)
- [ ] App Review Guidelines compliance verified: no private/undocumented API usage (iOS), no external payment links for digital goods (both platforms)
- [ ] AI/ML transparency disclosure added if app uses generative AI, LLMs, or external AI services (Apple & Google require disclosure)
- [ ] Login with Apple implemented if app offers any third-party social login (Apple requirement since 2020)
- [ ] App does not crash on launch or during primary flow — test the exact binary being submitted, not a debug build
- [ ] No placeholder content, test data, or "lorem ipsum" visible anywhere in the app

### Code Signing & Build Configuration

- [ ] iOS: Production distribution certificate and provisioning profile configured (not development/ad-hoc)
- [ ] Android: Release keystore created, backed up securely, and NOT committed to Git (losing the keystore = cannot update the app)
- [ ] Bundle ID / Application ID is unique, correctly formatted, and matches App Store Connect / Play Console entry
- [ ] Version number (`CFBundleShortVersionString` / `versionName`) and build number (`CFBundleVersion` / `versionCode`) incremented from last submission
- [ ] Release build tested on physical devices — not just emulator/simulator (test on oldest supported device you own)
- [ ] ProGuard / R8 code shrinking enabled for Android release builds (minifyEnabled true) — test that obfuscation doesn't break reflection/serialization
- [ ] Bitcode, dSYM, or debug symbols uploaded for crash symbolication (or configured to auto-upload via Crashlytics/Sentry)

### Security

- [ ] No API keys, secrets, tokens, or credentials hardcoded in source code or bundled in the app binary (extract with `strings` to verify)
- [ ] Sensitive API calls use certificate pinning or at minimum TLS 1.2+ (prevent MITM on public WiFi)
- [ ] User credentials and tokens encrypted at rest (iOS: Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`; Android: EncryptedSharedPreferences or Keystore)
- [ ] App Transport Security (ATS) is NOT globally disabled on iOS (`NSAllowsArbitraryLoads` must be `false` in production)
- [ ] Jailbreak/root detection implemented if app handles payments, banking, or sensitive health/financial data
- [ ] Sensitive screens disable screenshots/screen recording where required (iOS: window overlay; Android: `FLAG_SECURE`)
- [ ] Biometric authentication (Face ID/Touch ID, Android BiometricPrompt) used for sensitive actions if applicable

---

## 🟡 IMPORTANT

### Testing

- [ ] Tested on the minimum supported OS version declared in store listing (install on an old device or use Xcode/Android simulators)
- [ ] Tested across screen sizes: small (iPhone SE / Android compact), standard, large (Pro Max / tablet)
- [ ] Background → foreground transitions tested — no crash, stale data, or blank screen on app resume
- [ ] Push notifications tested end-to-end: delivery, display, tap → deep link to correct screen, permission prompt UX
- [ ] Offline state handled gracefully — show cached data or clear "no connection" message, never blank/frozen screens
- [ ] Network transitions tested: WiFi → cellular, cellular → offline, slow 3G simulation (use Network Link Conditioner / Charles Proxy)
- [ ] Memory pressure handling tested (iOS: `didReceiveMemoryWarning`; monitor with Instruments/Profiler for leaks)
- [ ] Deep links / universal links tested: tapping a link from email/browser opens the correct screen in the app
- [ ] App handles interrupted flows: incoming call during payment, backgrounded during upload, permission denied mid-flow
- [ ] Accessibility tested: VoiceOver (iOS) and TalkBack (Android) can navigate all primary flows
- [ ] Landscape orientation handled or explicitly locked to portrait (no half-rendered layouts)
- [ ] Force-kill and relaunch: app recovers state correctly, no data loss for in-progress work

### Store Listing & ASO (App Store Optimization)

- [ ] App icon: 1024×1024px PNG, no transparency/alpha channel (iOS strict requirement); 512×512px hi-res icon (Android)
- [ ] Screenshots prepared for all required device sizes (iOS: 6.7", 6.5", 5.5" at minimum; Android: phone + 7" + 10" tablet if supporting)
- [ ] App preview video prepared — 15-30 seconds showing core value prop (optional but significantly boosts conversion)
- [ ] App name/title optimized: ≤ 30 chars (Android) / ≤ 50 chars (iOS) — include primary search keyword
- [ ] Short description ≤ 80 chars (Android) — this is the most-read copy in the Play Store listing
- [ ] Full description keyword-optimized and clearly explains what the app does in the first 2 lines (most users don't expand)
- [ ] Content rating questionnaire completed accurately (wrong rating = removal risk)
- [ ] App category and subcategory chosen strategically (affects browse discoverability)
- [ ] What's New / Release Notes written — describe changes users care about, not internal refactors
- [ ] Subtitle (iOS) / Short description (Android) includes secondary keyword not in the title

### Performance & Monitoring

- [ ] Crash reporting SDK configured and verified — crashes appear in dashboard (Firebase Crashlytics, Sentry, Bugsnag)
- [ ] Analytics events tracking core funnel: app open, signup, key feature usage, purchase (Firebase Analytics, Amplitude, Mixpanel)
- [ ] App binary size optimized: < 50MB preferred for over-the-air install (use App Thinning on iOS; `bundletool` analysis on Android)
- [ ] Battery usage profiled — no excessive background CPU, GPS, or network drain (use Instruments Energy Log / Android Battery Profiler)
- [ ] Cold start time < 2 seconds on mid-range devices (profile with Instruments / Android Profiler; defer heavy init)
- [ ] Memory usage stays under 200MB during normal use (monitor for leaks with Instruments / LeakCanary)
- [ ] Network requests are efficient: batch where possible, paginate lists, compress payloads (gzip/brotli)
- [ ] Images and assets loaded at appropriate resolution for device screen density (@2x/@3x; mdpi through xxxhdpi)

### Compliance & Legal

- [ ] GDPR compliance: App Tracking Transparency (ATT) prompt shown on iOS 14.5+ before any tracking/advertising ID access
- [ ] CCPA compliance if serving California users: "Do Not Sell My Personal Information" option available if applicable
- [ ] Google Play Data Safety section filled out accurately (required — app will be flagged without it)
- [ ] Apple App Privacy nutrition labels filled out in App Store Connect (required — submission blocked without it)
- [ ] In-app purchases use platform IAP system for digital goods — no links to external payment (Apple/Google policy, enforced)
- [ ] Account deletion feature available if app offers account creation (Apple hard requirement; Google Play policy)
- [ ] Children's/COPPA compliance reviewed if app could attract under-13 users (age gate, restricted data collection)
- [ ] Data retention and deletion policies documented — user data is deletable upon request

---

## 🟢 NICE-TO-HAVE

- [ ] App Clip (iOS) or Instant App (Android) configured for lightweight trial experience without full install
- [ ] Localization for target markets: all user-facing strings externalized in `.strings` / `strings.xml` / ARB files
- [ ] Dark mode fully supported and tested (iOS: `@Environment(\.colorScheme)`; Android: `isNightMode`)
- [ ] Dynamic Type / font scaling support (iOS: use system text styles; Android: use `sp` units and test at largest scale)
- [ ] Home screen widgets configured for quick-glance information (WidgetKit on iOS; AppWidgetProvider on Android)
- [ ] Store listing A/B experiments set up in Google Play Console (test icon, screenshots, descriptions)
- [ ] In-app review prompt implemented at a moment of delight, not on first launch (StoreKit `requestReview` / Play In-App Review API)
- [ ] Force-update mechanism: remotely require users on critically broken versions to update (use Firebase Remote Config or custom API)
- [ ] Feature flags / remote config integrated for safe rollouts and kill switches (Firebase Remote Config, LaunchDarkly)
- [ ] Onboarding flow tested for first-time users — clear value prop, minimal friction to core experience
- [ ] Haptic feedback used thoughtfully for key interactions (success, error, selection)
- [ ] Adaptive icons configured for Android (foreground + background layers for consistent shape across launchers)
- [ ] iPad / tablet layout optimized if supporting larger screens (not just phone layout stretched)
