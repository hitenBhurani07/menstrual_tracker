# FEMCARE TECHNICAL INTERVIEW HANDBOOK
**Comprehensive Candidate Defense Guide & Architectural Blueprint**
*Target Interview Date: 2 November 2026 | Candidate: Hiten Bhurani*
*Repository: https://github.com/hitenbhurani/menstrual_tracker*

---

## EXECUTIVE VERIFICATION MATRIX: RESUME CLAIMS VS. REPOSITORY REALITY

Before entering any interview room, you must know the exact boundary between what is written on your resume and what physically exists in the codebase. Interviewers from top IT services companies and product teams immediately spot discrepancies between claims and code. Use this matrix as your compass of truth.

| Feature / Tech Claimed on Resume | Actual Status in Repository | Exact Evidence in Codebase | Truthful Spoken Explanation for Candidate |
| :--- | :--- | :--- | :--- |
| **Android Java Development** | **VERIFIED IMPLEMENTED** | `app/src/main/java/com/miniflo/femcare/` (39 Java source files, zero Kotlin) | *"The entire native mobile client is engineered in pure Java using Android SDK (compileSdk 36, minSdk 24) with Material Components."* |
| **Firebase Authentication** | **VERIFIED IMPLEMENTED** | `LoginActivity.java`, `RegisterActivity.java`, `AuthRepository.java` | *"Email/password registration and Google Sign-In via Google Play Services (`GoogleSignInClient`) and `FirebaseAuth` are fully functional."* |
| **Cloud Firestore** | **VERIFIED IMPLEMENTED** | `TrackFragment.java`, `CalendarFragment.java`, `TodayFragment.java`, `AuthRepository.java` | *"Used as the primary cloud database under document paths `users/{email}`, with subcollections `daily_logs`, `notes`, and `notifications`."* |
| **Room Database (SQLite)** | **VERIFIED IMPLEMENTED** | `AppDatabase.java`, `UserEntity.java`, `UserDao.java`, `CycleEntity.java`, `CycleDao.java` | *"Implemented using Room 2.6.1 with an SQLite database named `femcare_database`. Two tables exist: `user_table` and `cycle_table`."* |
| **Cycle Tracking & Predictions** | **VERIFIED IMPLEMENTED** | `TodayFragment.java` (`runPredictionAlgorithm`), `CalendarFragment.java` (`CalendarAdapter`) | *"Calculates next period date, ovulation day (cycleLength - 14), and fertile window using historical baseline math and a roll-forward while-loop."* |
| **Symptom & Mood Logging** | **VERIFIED IMPLEMENTED** | `TrackFragment.java` (`saveDailyLog`, `persistDailyLog`), `fragment_track.xml` | *"Logs symptoms, moods, vaginal discharge, energy levels, and hydration count into Firestore document `users/{email}/daily_logs/{yyyy-MM-dd}`."* |
| **Calendar Interface** | **VERIFIED IMPLEMENTED** | `CalendarFragment.java`, `GridLayoutManager(getContext(), 7)`, `GestureDetector` | *"Custom 7-column calendar grid with month navigation, swipe gestures, visual cycle phase rings, and daily note bottom-sheets."* |
| **WorkManager Reminders** | **VERIFIED IMPLEMENTED** | `BackgroundTaskScheduler.java`, `ScheduledNotificationWorker.java`, `DailyHealthWorker.java` | *"Enterprise-grade WorkManager engine scheduling periodic 24h daily reminders, 6h cycle checks, 7d wellness checks, and debounced immediate sync."* |
| **Node.js & Express Backend** | **PARTIALLY IMPLEMENTED / DECOUPLED** | `backend/server.js`, `backend/controllers/`, `backend/routes/` | *"A standalone Express API exists in the repo with CRUD endpoints, but it currently stores data in in-memory JavaScript arrays and is not connected to the Android app."* |
| **Retrofit / REST Client** | **CLAIMED BUT UNINSTANTIATED** | In `app/build.gradle.kts` dependencies, but zero Java imports or interface classes | *"Retrofit 2.9.0 and Gson dependencies were imported in Gradle to prepare for the Node.js backend migration, but network operations currently use Firebase SDK and Volley."* |
| **Volley Networking** | **VERIFIED IMPLEMENTED** | `FindDoctorActivity.java` (`JsonObjectRequest`, `RequestQueue`) | *"Used in FindDoctorActivity to query the OpenStreetMap Overpass REST API and Google Places API for nearby gynecological clinics."* |
| **Offline-First Synchronization** | **PARTIALLY IMPLEMENTED** | `LoginActivity.java` (`syncFirestoreToRoom`), `NotificationPublisher.java`, `LocalNotificationStore.java` | *"Read/cache synchronization works for user profile and notifications. Full bi-directional offline write queue with conflict resolution is a planned architecture."* |
| **Medical Report PDF Generation** | **VERIFIED IMPLEMENTED** | `SettingsFragment.java` (`createDetailedPdfReport`), `FileProvider` | *"Generates a multi-page PDF with canvas-drawn headers, watermarks, health metrics, and symptom tables, shared securely via Android FileProvider."* |

---

# PART 1 — FEMCARE FROM ZERO

### 1. What is FemCare?
FemCare is a native Android health and wellness application engineered specifically for biological females and individuals who menstruate. At its core, FemCare is a digital cycle diary, prediction engine, and reproductive health companion. It allows a user to register an account, record their biological baseline data (such as typical cycle length and period duration), log daily physiological and psychological symptoms, visualize upcoming cycle phases on an interactive calendar, and receive automated background notifications regarding period arrival, ovulation, and lifestyle habits.

### 2. What problem does it solve?
Menstruation is governed by a complex hormonal feedback loop involving estrogen, progesterone, luteinizing hormone (LH), and follicle-stimulating hormone (FSH). Because human memory is fallible, most individuals struggle to recall the exact start date of their previous menses, let alone detect subtle changes in cycle regularity or recognize symptom patterns (such as migraines occurring exclusively during the luteal phase). 

FemCare solves three acute problems:
1. **Unpredictability & Preparedness:** By applying date-arithmetic algorithms to historical cycle data, it eliminates the anxiety of unexpected menstruation.
2. **Clinical Symptom Correlation:** Users often experience bloating, mood shifts, cramps, or fatigue without connecting them to hormonal shifts. FemCare logs these daily, generating trend charts that can be shared with healthcare providers.
3. **Reproductive Intentions:** Whether a user is attempting to conceive (requiring precise identification of the fertile window) or avoiding pregnancy, FemCare highlights the estimated fertile phase and ovulation window.

### 3. Who are the intended users?
- **Adolescents and Young Adults:** Individuals seeking to understand their menstrual rhythms, learn their baseline cycle duration, and receive discrete reminders.
- **Individuals with Irregular Cycles or Conditions (e.g., PCOS, Thyroid disorders):** Users who require meticulous tracking of irregular cycles, stress fluctuations, and symptoms for medical evaluation.
- **Individuals Trying to Conceive (TTC):** Users who monitor ovulation days and the fertile window (5 days prior to ovulation plus ovulation day).
- **Working Professionals:** People who need timely notifications (e.g., "Period expected in 2 days") to manage their personal preparedness without manual calculation.

### 4. Why is a tracking application useful?
A mobile tracker transforms subjective human feelings into objective, structured, time-stamped clinical data:
- **Medical Diagnostics:** When consulting a gynecologist, the first question asked is, *"When was the first day of your last menstrual period (LMP)?"* FemCare provides this instantly.
- **Pattern Recognition:** By plotting mood and cramps against cycle days, users discover whether their anxiety is situational or a symptom of Premenstrual Dysphoric Disorder (PMDD).
- **Proactive Health Interventions:** Built-in intelligence alerts users when a period is late or when cycle lengths exceed 45 days (the clinical threshold for oligomenorrhea).

### 5. What information does a user enter?
During the **Onboarding Flow** (Activities chained sequentially):
1. **Date of Birth** (`BirthDateActivity`): Used to calculate age.
2. **Physical Demographics** (`UserInfoActivity`): Height (cm), Weight (kg) — the app automatically calculates Body Mass Index ($BMI = \frac{weight}{(height/100)^2}$), regularity perception, and birth control usage.
3. **Cycle Baselines** (`TypicalCycleActivity`): Typical cycle length in days (e.g., 28 days) and bleeding duration (e.g., 5 days).
4. **Last Period Date** (`LastPeriodActivity`): The start and end timestamps of their most recent menstrual bleed.
5. **Medical History** (`ReproductiveProblemsActivity`): Polycystic Ovary Syndrome (PCOS), thyroid disorders.
6. **Lifestyle Parameters** (`LifestyleActivity`): Pregnancy status, pregnancy attempts, average sleep hours, and weekly exercise frequency.
7. **Initial Symptoms** (`UneasinessActivity`): Baseline complaints like cramps, headaches, or acne.

During **Daily Usage** (`TrackFragment`):
- Physical Symptoms: Cramps, Headache, Acne, Backache, Nausea, Bloating, or custom symptoms.
- Moods: Happy, Sad, Anxious, Irritable, Mood Swings, or custom moods.
- Vaginal Discharge: Dry, Sticky, Creamy, Watery, Egg White.
- Energy Levels: High, Medium, Low, Exhausted.
- Hydration Count: Glasses of water consumed today (incremented/decremented).
- Period Status: Checkbox indicating whether bleeding has ended today.

### 6. What does the application calculate?
FemCare executes mathematical and algorithmic derivations:
1. **Age Calculation:** Evaluates user birth date against current calendar time.
2. **Body Mass Index (BMI):** Evaluated from height and weight; flags users under 18.5 (underweight) or over 25.0 (overweight) to trigger lifestyle wellness notifications.
3. **Next Estimated Period Start Date:** Takes the last recorded start timestamp and projects forward using the average cycle length.
4. **Cycle Roll-Forward Fix:** If the user has not opened the app for months and the last period timestamp is in the distant past, the algorithm steps forward cycle-by-cycle in a `while` loop until it locates the current or future cycle window.
5. **Ovulation Day:** Biologically estimated as 14 days *before* the next period start date ($Ovulation = NextPeriod - 14\text{ days}$).
6. **Fertile Window:** Calculated as the 5 days preceding ovulation through 1 day post-ovulation (a 7-day window where intercourse can result in conception due to sperm viability and ovum lifespan).
7. **Cycle Day Progress Percentage:** Formatted as a normalized float ($0.0$ to $1.0$) passed to a custom view (`HexagonProgressBar`) to visualize where the user currently sits in their cycle.
8. **Symptom Frequency Trends:** Aggregates past 30 days of daily log records and computes symptom frequency per day, feeding an MPAndroidChart `LineChart`.

### 7. What happens after a user records a symptom?
When a user selects chips in `TrackFragment` and taps **"Save Daily Log"**:
1. **Validation & Packaging:** Selected chip strings, water count, period end flag, and epoch timestamp are collected into a Java `HashMap<String, Object>`.
2. **Cloud Persistence:** The map is committed to Cloud Firestore at `users/{email}/daily_logs/{yyyy-MM-dd}` using `.set(logData, SetOptions.merge())`.
3. **Timeout Protection:** The network request is wrapped inside `wrapCloudSaveWithTimeout` (15-second fallback) to prevent UI freezing if connectivity drops.
4. **Local Feed Notification:** An entry is inserted into `LocalNotificationStore` (cached in SharedPreferences) so the in-app notification feed updates immediately.
5. **System Notification Trigger:** If enabled, an Android status bar notification is raised via `NotificationPublisher.publishForCurrentUser`.
6. **Background Worker Trigger:** `BackgroundTaskScheduler.enqueueImmediateSync(context, "daily_log_saved")` is dispatched to WorkManager to trigger downstream health checks.
7. **UI Refresh:** The daily symptom trend chart (`fetchSymptomTrends`) and historical calendar dot grid (`fetchPeriodLogicAndHistory`) automatically re-query and refresh.

### 8. Why have a calendar?
A calendar (`CalendarFragment`) is the universal mental model for time and health. In FemCare:
- It transforms abstract dates into a color-coded monthly grid (Crimson circle = active/expected period; Red dot = ovulation day; Pink halo = fertile window).
- It allows historical inspection: users can tap any previous day to view logged symptoms or add retroactive daily notes.
- It provides forward projection: users can swipe months ahead to plan vacations, athletic events, or medical procedures around their expected menstruation.

### 9. What are the main modules?
1. **Authentication & Onboarding Module:** Manages user creation, session persistence, and baseline medical demographic collection.
2. **Dashboard & Central Hub Module:** Hosts bottom navigation and displays the current daily cycle state, pregnancy chance, and hexagonal progress.
3. **Cycle Tracking & Symptom Logging Module:** Chip-based logging of multi-dimensional symptoms, water counter, and MPAndroidChart trend visualization.
4. **Interactive Cycle Calendar Module:** Custom 7-column grid with swipe gesture recognition and bottom-sheet daily notes.
5. **Local & Cloud Persistence Module:** Dual-layer architecture combining Room SQLite (`femcare_database`) and Cloud Firestore.
6. **Background Automation & Notification Engine:** WorkManager workers executing periodic and event-driven health evaluations.
7. **Medical Documents & Clinical Export Module:** MediaStore image capture, file picker, and on-device native PDF document compiler.
8. **Nearby Healthcare Locator Module:** Location-aware finder querying OpenStreetMap Overpass and Google Places APIs with osmdroid rendering.

### 10. What are the limitations of cycle prediction?
- **Biological Non-Determinism:** Mathematical prediction assumes hormonal regularity. Stress, travel, illness, hormonal medication, polycystic ovaries, and thyroid fluctuations alter follicular phase duration unpredictably.
- **Fixed Luteal Phase Assumption:** FemCare assumes a standard 14-day luteal phase ($Ovulation = NextPeriod - 14$). In real biology, the luteal phase varies between 10 and 16 days across individuals.
- **No Direct Biomarkers:** FemCare does not measure physiological biomarkers (such as Basal Body Temperature [BBT], urinary Luteinizing Hormone [LH] surges, or cervical mucus elasticity). It is an arithmetic estimator, not a clinical diagnostic tool.
- **Cold-Start Problem:** For new users with only one recorded period, predictions rely on a static baseline average (e.g., 28 days) rather than an individual moving average.

---

# PART 2 — PROJECT EXPLANATION SCRIPTS

### 30-Second Explanation (The Elevator Pitch)
> *"FemCare is a native Android women's health application built in Java that helps users track menstrual cycles, predict upcoming periods, and log daily physiological symptoms. Architecturally, it utilizes a hybrid data model: Firebase Cloud Firestore for real-time cloud backup and multi-device sync, and Room SQLite for instant offline access. It features an automated background engine powered by Android WorkManager that calculates period and ovulation alerts without draining device battery, and includes a medical report export module that compiles health histories into encrypted PDF documents for gynecological consultations."*

### 60-Second Explanation (Standard Interview Introduction)
> *"FemCare is a comprehensive menstrual health and symptom tracking application developed natively for Android using Java. I built it to solve the problem of cycle unpredictability and unstructured medical logging that many women experience.*
>
> *The core technical highlight of FemCare is its resilient hybrid data architecture and background scheduling. When a user registers, their health profile and cycle history are stored in Google Cloud Firestore and simultaneously cached locally in an SQLite database using the Room Persistence Library. This ensures the app is fully usable offline.*
>
> *For background intelligence, rather than relying on unreliable timers, I architected a background system using Android Jetpack's WorkManager API. It schedules periodic tasks for daily log reminders, 6-hour cycle milestone evaluations, and weekly wellness notifications, enforcing system constraints like battery health and network connectivity.*
>
> *The app also integrates MPAndroidChart for symptom trend analytics, Android's native PdfDocument API for clinical PDF generation, and a location-based clinic finder combining OpenStreetMap and Google APIs."*

### 2-Minute Explanation (Deep Technical Architectural Overview)
> *"FemCare is a native Android menstrual cycle tracking and predictive health platform engineered in Java. The project was conceived to address both everyday cycle tracking and long-term medical symptom documentation.*
>
> *From an architectural standpoint, the application follows an Activity-Fragment modular structure with an MVVM-inspired data access layer. The user begins with a guided onboarding wizard spanning seven dedicated activities that capture baseline metrics such as cycle duration, bleeding length, BMI, and conditions like PCOS. This data is written through an AuthRepository to Cloud Firestore under user-scoped document paths, and mirrored into Room SQLite entities (`UserEntity` and `CycleEntity`).*
>
> *On the front-end, `TodayFragment` acts as the command center. It implements a roll-forward cycle prediction algorithm that determines the current cycle day, fertile window, and ovulation day, rendering cycle completion on a custom canvas view called `HexagonProgressBar`. The `TrackFragment` enables multi-symptom logging across moods, physical sensations, discharge, and water intake, instantly plotting 30-day symptom frequency trends on a line chart.*
>
> *One of the most complex parts of the system is the background automation engine. In `BackgroundTaskScheduler`, I configured WorkManager with `PeriodicWorkRequest` and `OneTimeWorkRequest` policies. Periodic workers check every 24 hours at 8:00 PM for missed logs and every 6 hours for cycle transitions. To prevent background churn during Firebase token refresh degradation, I built a custom `FirebaseAuthState` detector that implements exponential backoff and defers background sync tasks by 2 minutes if auth errors occur.*
>
> *Finally, the application features an offline-first notification cache (`LocalNotificationStore`), on-device clinical PDF compilation using `PdfDocument` shared securely via `FileProvider`, and a nearby clinic locator using `osmdroid` and `Volley`. In the repository, you will also find a modular Node.js/Express backend that I developed for API experimentation."*

---

### Key Interview Questions & Natural Spoken Responses

#### "Explain your project."
*"FemCare is an Android application written in Java that combines menstrual cycle prediction, multi-category symptom logging, offline persistence, and background notification scheduling. It acts as an intelligent health companion that turns subjective daily feelings into structured health data."*

#### "Why did you choose this project?"
*"I chose this domain because menstrual tracking is a mission-critical, data-sensitive problem where reliability and offline functionality are paramount. A user cannot afford to lose their health data simply because they are in an area with poor cellular coverage. From an engineering perspective, it presented the perfect challenge to combine date arithmetic, local database persistence with Room, cloud synchronization with Firebase, and background execution using Android WorkManager."*

#### "What was the problem statement?"
*"The problem statement was twofold: first, the lack of accessible, private, and automated tools for women to track menstrual irregularities and fertility phases; and second, the technical challenge of building an Android mobile application that provides real-time cloud synchronization without sacrificing offline usability or draining device battery with persistent background services."*

#### "What is the most important feature?"
*"The most critical feature is the cycle prediction engine integrated with the background scheduling system. The mathematical algorithm calculates the next period start date, the 7-day fertile window, and ovulation day. But more importantly, the WorkManager background worker evaluates this math continuously in the background, alerting the user 2 days prior to period onset and on their ovulation day even if the app process has been killed."*

#### "What makes your application useful?"
*"It bridges the gap between daily lifestyle habits and clinical diagnostics. Instead of trying to remember when symptoms occurred, a user can hand their doctor a cleanly compiled, multi-page PDF generated directly by the app containing exact symptom tables, water intake trends, BMI metrics, and cycle variance over the past 30 days."*

#### "What is the most challenging technical part?"
*"The single most challenging technical aspect was managing Firebase authentication token stability and cloud write timeouts in unstable network conditions. During network fluctuations, Firestore write listeners could hang or throw secure-token exceptions, causing UI lag. I solved this by engineering a custom `FirebaseAuthState` backoff handler that intercepts token exceptions, defers background work, and applies a 15-second safety timeout wrapper (`wrapCloudSaveWithTimeout`) around UI save buttons so the user is never left with an unresponsive screen."*

---

# PART 3 — COMPLETE ANDROID ARCHITECTURE

### High-Level System Architecture Diagram

```
+----------------------------------------------------------------------------------------------------+
|                                    PRESENTATION LAYER (UI)                                         |
|                                                                                                    |
|  [ MainActivity ] --------> [ LoginActivity / RegisterActivity ]                                  |
|         |                                  |                                                       |
|   (Splash & Perms)                         v                                                       |
|         |                   [ 7-Step Onboarding Activities ]                                       |
|         |            (BirthDate -> UserInfo -> TypicalCycle -> LastPeriod                          |
|         |             -> ReproductiveProblems -> Lifestyle -> Uneasiness)                          |
|         v                                  |                                                       |
|  [ DashboardActivity ] <-------------------+                                                       |
|    |--> BottomNavigationView + FloatingActionButton                                                |
|    |--> FragmentContainerView                                                                      |
|           |---> [ TodayFragment ]        : Cycle hub, HexagonProgressBar, Roll-forward math        |
|           |---> [ CalendarFragment ]     : 7xN GridLayout, Swipe Gestures, Date Notes BottomSheet  |
|           |---> [ TrackFragment ]        : Symptom Chips, Water Counter, MPAndroidChart Trends     |
|           |---> [ NotificationsFragment ]: Merged Cloud & Local Notification Feed                  |
|           |---> [ SettingsFragment ]     : Profile, Reminders, Night Mode, Native PDF Generation   |
|                                                                                                    |
|  [ FindDoctorActivity ]    : OSM MapView (osmdroid) + Google Places (Volley) + Navigation Intent   |
|  [ MedicalReportActivity ] : MediaStore Camera / Storage Picker -> Local Report Storage            |
+----------------------------------------------------------------------------------------------------+
                                             |           ^
                                             v           |
+----------------------------------------------------------------------------------------------------+
|                                 APPLICATION & CONTROLLER LAYER                                     |
|                                                                                                    |
|  [ FemCareApplication ]       : Global app init, Room DB warm-up, UncaughtExceptionHandler restart |
|  [ AuthViewModel ]            : AndroidViewModel exposing LiveData to Activities                  |
|  [ AuthRepository ]           : Data broker coordinating Cloud Firestore & Room SQLite             |
|  [ FirebaseAuthState ]        : Detects token degradation, backoff logic, error dialogs           |
|  [ FirebaseConnectionMonitor ]: Realtime DB connection state watcher                               |
+----------------------------------------------------------------------------------------------------+
                                             |           ^
                                             v           |
+----------------------------------------------------------------------------------------------------+
|                                  BACKGROUND EXECUTION LAYER                                        |
|                                                                                                    |
|  [ BackgroundTaskScheduler ]  : Central WorkManager dispatcher (Periodic: 24h, 6h, 7d; Immediate)  |
|  [ ScheduledNotificationWorker]: Multi-mode execution engine (MODE_DAILY, CYCLE, WEEKLY, IMMEDIATE)|
|  [ DailyHealthWorker ]        : Evaluates daily log completion and 2-day period warnings           |
|  [ PeriodReminderWorker ]     : Queries Room CycleDao to evaluate 1-day advance period alerts      |
|  [ MyFirebaseMessagingService]: FCM token capture, SharedPreferences cache, RemoteMessage receive  |
|  [ NotificationPublisher ]    : Unified gateway writing Firestore notifs & Android system channels |
+----------------------------------------------------------------------------------------------------+
                       |                                                     |
                       v                                                     v
+---------------------------------------------+       +----------------------------------------------+
|             LOCAL PERSISTENCE               |       |               CLOUD PERSISTENCE              |
|                                             |       |                                              |
|  [ Room Database: femcare_database ]        |       |  [ Firebase Cloud Firestore ]                |
|    - UserEntity   <-> UserDao               |       |    - Collection: "users/{email}"             |
|    - CycleEntity  <-> CycleDao              |       |    - Subcollection: "daily_logs/{dateKey}"   |
|                                             |       |    - Subcollection: "notes/{dateKey}"        |
|  [ SharedPreferences: FemCarePrefs ]        |       |    - Subcollection: "notifications/{notifId}"|
|    - Onboarding flags, dark mode, FCM token |       |                                              |
|                                             |       |  [ Firebase Authentication ]                 |
|  [ LocalNotificationStore (JSON in Prefs) ] |       |    - Email/Password Provider                 |
|    - Offline notification feed cache        |       |    - Google Sign-In Provider (OAuth2)        |
|                                             |       |                                              |
|  [ App External File Storage ]              |       |  [ External REST APIs ]                      |
|    - Captured reports & generated PDF docs  |       |    - OSM Overpass API (Clinic discovery)     |
|    - Exposed via Android FileProvider       |       |    - Google Places API (Ratings & Reviews)   |
+---------------------------------------------+       +----------------------------------------------+
```

---

### Component-by-Component Mapping & Verification

#### 1. Activities (`android.app.Activity`)
Activities represent distinct screens with focused user tasks. In FemCare, there are 13 activities:
- `MainActivity.java`: Splash screen. Evaluates permissions (`Manifest.permission.POST_NOTIFICATIONS`) and inspects session state (`FirebaseAuth.getCurrentUser()` and `onboarding_complete` flag) to determine navigation routing.
- `LoginActivity.java`: Authenticates users via email/password or Google Play Services. Upon success, invokes `syncFirestoreToRoom()` and kicks off `BackgroundTaskScheduler.scheduleAll()`.
- `RegisterActivity.java`: Creates new user credentials via `AuthViewModel.register()` and commits baseline document to Firestore before routing to onboarding.
- Onboarding Wizard (`BirthDateActivity`, `UserInfoActivity`, `TypicalCycleActivity`, `LastPeriodActivity`, `ReproductiveProblemsActivity`, `LifestyleActivity`, `UneasinessActivity`): Sequentially collect user health parameters, using `AuthViewModel` to persist fields to both Firestore and Room.
- `DashboardActivity.java`: The main navigation shell. Hosts `BottomNavigationView`, manages fragment transactions into `R.id.fragment_container`, and handles deep links from system notifications (`NotificationHelper.EXTRA_OPEN_NOTIFICATIONS`).
- `MedicalReportActivity.java`: Manages camera capture (`MediaStore.ACTION_IMAGE_CAPTURE`) and document picking (`Intent.ACTION_GET_CONTENT`), saving files to external app storage.
- `FindDoctorActivity.java`: Coordinates map display (`org.osmdroid.views.MapView`), device geolocation (`FusedLocationProviderClient`), and Volley HTTP requests.

#### 2. Fragments (`androidx.fragment.app.Fragment`)
Fragments represent modular, reusable portions of UI hosted within `DashboardActivity`:
- `TodayFragment.java`: The landing screen. Fetches cycle baselines, runs the prediction math, drives `HexagonProgressBar`, and displays personalized insights.
- `CalendarFragment.java`: Displays the monthly cycle grid using `GridLayoutManager(getContext(), 7)`, handles swipe gestures, and opens a `BottomSheetDialog` to save daily notes.
- `TrackFragment.java`: Full-screen logging form opened via FAB. Manages chip selections for symptoms, moods, discharge, energy, water counters, and renders the 30-day `LineChart`.
- `NotificationsFragment.java`: Loads local notification feed from `LocalNotificationStore`, attaches a real-time Firestore snapshot listener, and renders items in a `RecyclerView`.
- `SettingsFragment.java`: Manages reminder toggles, dark mode switching, user profile editing, session sign-out, and triggers native PDF medical report generation.

#### 3. XML Layouts & Views
- Material Components: `com.google.android.material.button.MaterialButton`, `TextInputEditText`, `TextInputLayout`, `SwitchMaterial`, `Chip`.
- Custom Views: `HexagonProgressBar.java` overrides `onDraw(Canvas canvas)` to render a geometric hexagonal cycle progress indicator using `Path` and `Paint`.
- Layout Managers: `ConstraintLayout` for responsive UI, `GridLayoutManager` (7 columns) for calendar grids, and `LinearLayoutManager` for notification and note lists.

#### 4. The Truth About Architecture: MVVM vs. Repository Pattern
In an interview, you must be 100% accurate regarding what is actually implemented:
- **Verified MVVM Implementation:** Present in the **Authentication & Onboarding module**. You have `UserEntity`/`CycleEntity` (Model), `activity_*.xml` (View), `AuthViewModel.java` extending `AndroidViewModel` (ViewModel), and `AuthRepository.java` (Repository broker). LiveData is used to observe registration and onboarding status.
- **Direct Controller Implementation:** In `TodayFragment`, `CalendarFragment`, `TrackFragment`, and `SettingsFragment`, the code directly invokes `FirebaseFirestore.getInstance()` and manages asynchronous callbacks inside the Fragment. 
- **The Ideal / Recommended Alternative (How you would improve it):** *"While the onboarding flow strictly adheres to MVVM with `AuthViewModel` and `AuthRepository`, the core tracking fragments currently query Firestore directly. In a production refactor, I would abstract `TrackFragment` and `CalendarFragment` behind a `CycleViewModel` and `CycleRepository`, exposing immutable `LiveData` or Kotlin `StateFlow` to decouple UI rendering from network/database callbacks."*

---

# PART 4 — ANDROID AND JAVA FUNDAMENTALS

### 17 Essential Android SDK Concepts

| # | Concept | Used in FemCare? | Exact Purpose & Location in Codebase |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **Android OS** | Foundation | The Linux-based open-source mobile operating system powering target devices. |
| 2 | **Android SDK** | Foundation | Software Development Kit providing Android APIs, tools, and compilers (compiled against SDK 36). |
| 3 | **Activity** | **USED** | Single focused screen. FemCare has 13 activities (`MainActivity`, `DashboardActivity`, etc.). |
| 4 | **Fragment** | **USED** | Sub-activity modular UI component. FemCare has 5 fragments swapped inside `DashboardActivity`. |
| 5 | **Intent** | **USED** | Message object used to start activities (`new Intent(this, UserInfoActivity.class)`) or trigger external apps (`Intent.ACTION_VIEW` for Google Maps). |
| 6 | **XML Layout** | **USED** | Declarative UI markup defining widget hierarchies (e.g., `activity_dashboard.xml`, `fragment_track.xml`). |
| 7 | **View** | **USED** | Basic UI building block (`TextView`, `Button`, `ImageView`, and custom `HexagonProgressBar`). |
| 8 | **Event Listener** | **USED** | Interfaces capturing user interactions (e.g., `setOnClickListener`, `setOnCheckedChangeListener`). |
| 9 | **Activity Lifecycle** | **USED** | State machine (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`). Used in `FindDoctorActivity` to pause/resume map rendering. |
| 10| **Context** | **USED** | Interface to global application environment. Used for resource loading, database instantiation (`AppDatabase.getInstance(context)`), and system services. |
| 11| **SharedPreferences**| **USED** | Key-value XML storage for lightweight state: `onboarding_complete`, `pref_dark_mode`, `fcm_token`, and notification JSON feed. |
| 12| **RecyclerView** | **USED** | Memory-efficient scrolling container recycling item views. Used for calendar cells, notes, and notification lists. |
| 13| **Adapter** | **USED** | Bridge binding data collections to `RecyclerView.ViewHolder` instances (`CalendarAdapter`, `NotifAdapter`, `ClinicAdapter`). |
| 14| **Background Task**| **USED** | Operations executing off the main thread. Orchestrated via `WorkManager`, `ExecutorService`, and `Thread`. |
| 15| **Permissions** | **USED** | Runtime security model. FemCare requests `POST_NOTIFICATIONS`, `ACCESS_FINE_LOCATION`, `CAMERA`, and storage permissions. |
| 16| **APK** | Concept | Android Package file format (bytecode, resources, manifest) distributed to devices. |
| 17| **Gradle** | **USED** | Build automation tool. Managed via Kotlin DSL (`build.gradle.kts`) handling dependencies and SDK configuration. |

---

### Core Java OOP Concepts Applied to FemCare

#### 1. Classes & Objects
- **Class:** The blueprint defining state and behavior.
  - *FemCare Example:* `Clinic.java` is a class defining properties of a medical center (`name`, `address`, `rating`, `distanceKm`).
- **Object:** A concrete instance allocated in heap memory.
  - *FemCare Example:* In `FindDoctorActivity.java`, `Clinic clinic = new Clinic(...)` creates an individual clinic object parsed from JSON.

#### 2. Constructors
- A special method invoked during object instantiation to initialize state.
  - *FemCare Example:* In `CycleEntity.java`:
    ```java
    public CycleEntity(long startDateMillis, long endDateMillis, int cycleLength, int periodDuration) {
        this.startDateMillis = startDateMillis;
        this.endDateMillis = endDateMillis;
        this.cycleLength = cycleLength;
        this.periodDuration = periodDuration;
        this.isIrregular = cycleLength > 45; // Enforces business validation rule at construction!
    }
    ```

#### 3. Inheritance
- The mechanism where a subclass acquires fields and methods of a parent class using `extends`.
  - *FemCare Example:* `TodayFragment extends Fragment` inherits lifecycle callbacks (`onCreateView`, `onDestroy`). `ScheduledNotificationWorker extends Worker` inherits background execution contracts from Jetpack WorkManager.

#### 4. Interfaces
- A contract defining abstract methods that implementing classes must fulfill (`implements`).
  - *FemCare Example:* In `CycleDao.java`:
    ```java
    @Dao
    public interface CycleDao {
        @Insert
        void insertCycle(CycleEntity cycle);
        @Query("SELECT * FROM cycle_table ORDER BY startDateMillis DESC")
        LiveData<List<CycleEntity>> getAllCycles();
    }
    ```
    Room generates the concrete underlying SQLite implementation class at compile time.

#### 5. Encapsulation
- Bundling data and methods within a single unit while restricting direct external access via access modifiers and getter/setter methods.
  - *FemCare Example:* In `Clinic.java`, fields (`private String name`, `private float rating`) are hidden, exposed only through `getName()` and `setRating()`.

#### 6. Polymorphism
- The ability for different classes to respond to the same message in distinct ways (Overriding and Overloading).
  - *Method Overriding:* In `ScheduledNotificationWorker.java`, `@Override public Result doWork()` replaces the default worker implementation with custom health notification evaluation.
  - *Method Overloading:* In `NotificationHelper.java`, `showNotification(context, title, msg)` overloads `showNotification(context, notifId, title, msg)`.

#### 7. Collections Framework
- High-performance data structures (`List`, `ArrayList`, `Map`, `HashMap`).
  - *FemCare Example:* `TrackFragment` maintains `private List<String> selectedLogs = new ArrayList<>()` to track active symptom chips, and a `Map<String, Object> logData = new HashMap<>()` to package documents for Firestore.

#### 8. Exception Handling
- Defensive programming via `try-catch-finally` to ensure application resilience.
  - *FemCare Example:* In `FemCareApplication.java`, a global `Thread.setDefaultUncaughtExceptionHandler` catches uncaught runtime exceptions, writes a crash dump to `SharedPreferences`, and schedules an `AlarmManager` process restart before terminating safely.

#### 9. Access Modifiers
- `public`: Accessible anywhere (e.g., `AppDatabase.getInstance()`).
- `private`: Accessible only within the declaring class (e.g., `BackgroundTaskScheduler` constructor is `private` to prevent instantiation of utility class).
- `protected`: Accessible within the package and subclasses (e.g., `onCreate(Bundle savedInstanceState)` in Activities).
- *Package-private (default):* Accessible within `com.miniflo.femcare` package.


---

# PART 5 — FIREBASE COMPLETE DEEP DIVE

### 1. Firebase Authentication Deep Dive
Authentication is the process of verifying *who* a user is, whereas authorization determines *what* resources that authenticated user is permitted to access. In FemCare, Firebase Authentication serves as the identity provider, abstracting away the security vulnerabilities of storing raw passwords or managing cryptographic salting/hashing algorithms on a custom server.

#### Registration & Login Flow in FemCare:
1. **Credential Submission:** In `RegisterActivity.java` or `LoginActivity.java`, the user provides their email and password or clicks the Google Sign-In button.
2. **Identity Handshake:** 
   - *Email/Password:* `FirebaseAuth.getInstance().createUserWithEmailAndPassword(email, password)` or `signInWithEmailAndPassword(email, password)` sends an encrypted HTTPS request to Google's Identity Toolkit backend.
   - *Google Sign-In:* Uses Google Play Services (`GoogleSignInClient`). Upon successful OAuth2 consent, a `GoogleSignInAccount` is returned. Its `idToken` is converted into a Firebase credential via `GoogleAuthProvider.getCredential(acct.getIdToken(), null)` and passed to `mAuth.signInWithCredential(credential)`.
3. **Identity Token Generation:** Firebase validates the credentials and returns a `FirebaseUser` object containing:
   - Unique User Identifier (`UID`).
   - Verified email address.
   - JSON Web Token (JWT) identity token and a long-lived Refresh Token.
4. **Session Persistence:** The Firebase Android SDK automatically persists the user's session tokens in encrypted local app storage. When the app is killed and restarted, `FirebaseAuth.getInstance().getCurrentUser()` immediately returns the logged-in user without re-prompting for credentials.
5. **Token Refresh Lifecycle:** The JWT access token expires every 60 minutes. The Firebase SDK automatically communicates with Google's Secure Token service in the background to refresh the access token using the stored refresh token.

---

### 2. Cloud Firestore Architecture

Cloud Firestore is a flexible, scalable NoSQL document-oriented cloud database hosted on Google Cloud Platform. Data is stored in **Documents**, which are grouped into **Collections**. Documents contain key-value fields and can also host **Subcollections**.

#### FemCare Document Hierarchy:
```
firestore_root
  └── users (Collection)
        └── hiten@example.com (Document: User Profile)
              ├── email: "hiten@example.com" (String)
              ├── name: "Hiten Bhurani" (String)
              ├── age: 24 (Number)
              ├── averageCycleLength: 28 (Number)
              ├── periodDuration: 5 (Number)
              ├── lastPeriodStartMillis: 1727740800000 (Number)
              ├── lastPeriodEndMillis: 1728172800000 (Number)
              ├── heightCm: 165.0 (Number)
              ├── weightKg: 58.0 (Number)
              ├── bmi: 21.3 (Number)
              ├── isRegular: true (Boolean)
              ├── hasPCOS: false (Boolean)
              ├── hasThyroid: false (Boolean)
              ├── onBirthControl: false (Boolean)
              ├── isPregnant: false (Boolean)
              ├── tryingToConceive: false (Boolean)
              ├── sleepHours: 8 (Number)
              ├── exerciseFrequency: 4 (Number)
              ├── onboardingComplete: true (Boolean)
              ├── fcm_token: "dK9..." (String)
              │
              ├── daily_logs (Subcollection)
              │     └── 2026-10-30 (Document: Daily Log Entry)
              │           ├── date: "2026-10-30" (String)
              │           ├── timestamp: 1730260800000 (Number)
              │           ├── waterCount: 8 (Number)
              │           ├── periodEnds: false (Boolean)
              │           └── symptoms: ["Cramps", "Bloating", "Happy", "Egg White"] (Array)
              │
              ├── notes (Subcollection)
              │     └── 2026-10-30 (Document: Calendar Note)
              │           ├── text: "Mild cramping started in afternoon" (String)
              │           ├── cycleStatus: "Ovulation Day" (String)
              │           └── timestamp: 1730261200000 (Number)
              │
              └── notifications (Subcollection)
                    └── notif_1730260800 (Document: Notification Feed Item)
                          ├── title: "Period Reminder" (String)
                          ├── message: "Your cycle starts in 2 days." (String)
                          ├── type: "reminder" (String)
                          ├── isRead: false (Boolean)
                          └── timestamp: 1730260800000 (Number)
```

---

### Realistic JSON Payloads

#### User Profile Document (`users/hiten@example.com`)
```json
{
  "email": "hiten@example.com",
  "name": "Hiten Bhurani",
  "age": 24,
  "averageCycleLength": 28,
  "periodDuration": 5,
  "lastPeriodStartMillis": 1727740800000,
  "lastPeriodEndMillis": 1728172800000,
  "heightCm": 165.0,
  "weightKg": 58.0,
  "bmi": 21.3,
  "isRegular": true,
  "hasPCOS": false,
  "hasThyroid": false,
  "onBirthControl": false,
  "isPregnant": false,
  "tryingToConceive": false,
  "sleepHours": 8,
  "exerciseFrequency": 4,
  "symptoms": "Cramps, Headache",
  "onboardingComplete": true,
  "fcm_token": "eX7z_fcm_sample_token_alpha_9812",
  "fcm_token_updated_at": 1730260800000
}
```

#### Symptom & Log Document (`users/hiten@example.com/daily_logs/2026-10-30`)
```json
{
  "date": "2026-10-30",
  "timestamp": 1730260800000,
  "waterCount": 7,
  "periodEnds": false,
  "symptoms": [
    "Cramps",
    "Bloating",
    "Irritable",
    "Creamy Discharge",
    "Medium Energy"
  ]
}
```

---

### Database & Backend Comparisons

| Architectural Dimension | Cloud Firestore | Firebase Realtime Database | Custom Node.js / Express Backend |
| :--- | :--- | :--- | :--- |
| **Data Model** | Document-Collection (NoSQL) | Single monolithic JSON tree | Relational (SQL) or Document (Mongo) |
| **Querying Capability**| Rich compound queries, shallow queries, multi-field indexing | Shallow queries not supported; filtering is deeply limited | Unlimited complex SQL JOINs, aggregations, GROUP BY |
| **Offline Support** | Fully automated local persistence & cache sync | Basic disk persistence | Manual caching required (Room / Retrofit) |
| **Scaling** | Automatic horizontal sharding across Google Cloud | Limited to ~200k concurrent connections per DB instance | Requires load balancers, auto-scaling groups, Redis |
| **Security Model** | Declarative Firestore Security Rules | Realtime Database Rules | Middleware, JWT validation, RBAC, OAuth2 |
| **Pricing Model** | Charged per document Read / Write / Delete | Charged per GB data stored & bandwidth transferred | Fixed compute server costs (VMs / Containers) |

#### Firestore vs. MySQL / Relational Databases:
- **Schema Flexibility:** Firestore is schema-less. You can add new fields (`waterCount`, `fcm_token`) without executing database schema migrations (`ALTER TABLE`). MySQL requires strict declarative schemas and migrations.
- **Relational Integrity:** MySQL enforces Foreign Keys (`FOREIGN KEY`) and Referential Integrity (cascading deletes). Firestore does not support foreign keys; data relationships must be managed programmatically.
- **Transactions:** MySQL provides ACID transactions across arbitrary tables. Firestore supports ACID transactions only within localized document sets (max 500 documents per batch).

---

### Firebase Security Rules & Vulnerability Analysis

In mobile applications communicating directly with Cloud Firestore without an intermediary backend server, **Firestore Security Rules are the ONLY barrier preventing unauthorized data access or malicious overwriting.**

#### Catastrophic Insecure Rule (Common Rookie Blunder):
```javascript
// CATASTROPHIC: Allows anyone in the world to read, modify, or wipe out every user's medical logs!
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true; 
    }
  }
}
```

#### Production-Grade Enforced Security Rule for FemCare:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Helper function checking if request is authenticated
    function isAuthenticated() {
      return request.auth != null;
    }
    
    // Enforce that a user can ONLY read and write their OWN document tree
    match /users/{userEmail} {
      allow read, write: if isAuthenticated() && request.auth.token.email == userEmail;
      
      // Cascade ownership check to all subcollections (daily_logs, notes, notifications)
      match /daily_logs/{logDate} {
        allow read, write: if isAuthenticated() && request.auth.token.email == userEmail;
      }
      match /notes/{noteDate} {
        allow read, write: if isAuthenticated() && request.auth.token.email == userEmail;
      }
      match /notifications/{notifId} {
        allow read, write: if isAuthenticated() && request.auth.token.email == userEmail;
      }
    }
  }
}
```

#### Security Risks in FemCare & How to Defend:
- **Email as Document Key:** FemCare currently uses the user's plain text email (`users/{email}`) as the document key.
  - *Risk:* Email addresses can change, and special characters (`.`, `@`) can complicate URL path routing.
  - *Interview Defense:* *"In production, using `request.auth.uid` as the document key (`users/{uid}`) is superior to email. UID is immutable, cryptographically generated, and decouples user identity from their contact information."*

---

# PART 6 — ROOM AND SQLITE DEEP DIVE

### 12 Fundamental Local Database Questions Answered

#### 1. What is SQLite?
SQLite is a lightweight, serverless, self-contained, transactional SQL database engine embedded directly into the Android operating system. Unlike client-server databases like PostgreSQL or MySQL, SQLite reads and writes directly to regular disk files in the app's private sandbox directory (`/data/data/com.miniflo.femcare/databases/`).

#### 2. What is Room?
Room is an Android Jetpack persistence library that provides an abstraction layer over SQLite. It eliminates boilerplate code (such as `SQLiteOpenHelper`, manual `Cursor` indexing, and raw SQL string queries) and provides compile-time verification of SQL queries.

#### 3. Why do Android applications use local databases?
- **Zero-Latency Data Access:** Disk reads from a local SQLite database execute in milliseconds, providing instant 60 FPS UI rendering without network spinners.
- **Offline Reliability:** Users can view their past cycles, log daily symptoms, and inspect calendar dates on airplanes or in basements without cellular reception.
- **Bandwidth & Battery Conservation:** Prevents redundant network calls to Firestore on every screen transition.

#### 4. What is an Entity?
An Entity is a plain Java model class annotated with `@Entity` that represents a table within the SQLite database. Each field in the class represents a column in the table.

#### 5. What is a DAO (Data Access Object)?
A DAO is an interface annotated with `@Dao` that defines the contract for database interactions. It maps Java method calls directly to SQL operations (`@Insert`, `@Update`, `@Delete`, `@Query`).

#### 6. What is a Database class?
An abstract class annotated with `@Database` that extends `RoomDatabase`. It defines the list of associated entities, the schema version number, and exposes abstract getter methods for each DAO.

#### 7. What is a primary key?
A primary key (`@PrimaryKey`) is a unique identifier for each row in a database table. It guarantees that no two rows can share the exact same key, enforcing uniqueness.

#### 8. What is a foreign key?
A constraint linking a column in one table to the primary key of another table (`@ForeignKey`). It prevents orphaned records and enforces referential integrity.

#### 9. What are indexes?
An index is a database data structure (typically a B-Tree) that accelerates data retrieval on specific columns at the cost of additional disk space and slightly slower write speeds.

#### 10. What is a database migration?
When the database schema changes (e.g., adding a column like `waterGlasses` to `CycleEntity`), Room requires a `Migration` object detailing the SQL commands (`ALTER TABLE`) to transition from version $N$ to $N+1$ without destroying user data. If migrations are not supplied, calling `.fallbackToDestructiveMigration()` wipes the database clean.

#### 11. What is CRUD?
CRUD stands for the four foundational operations of persistent storage: **C**reate (`@Insert`), **R**ead (`@Query`), **U**pdate (`@Update`), and **D**elete (`@Delete`).

#### 12. What is offline persistence?
The ability of an application to cache operational data locally, allow users to continue reading and mutating state without network connectivity, and maintain consistency until synchronization can occur.

---

### Room is an Abstraction Layer, Not a Replacement

> **Critical Interview Distinction:** Room does **NOT** replace SQLite. SQLite is the actual native database engine running in C inside Android's Linux kernel. Room is an Object-Relational Mapping (ORM) compiler that translates Java method invocations into SQLite SQL queries, converts `Cursor` rows into Java POJOs, and verifies SQL syntax at compile time.

```
+-------------------------------------------------------------------+
|                        APPLICATION CODE                           |
|      AuthRepository.java  /  PeriodReminderWorker.java            |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                     ROOM PERSISTENCE LIBRARY                      |
|   - Type Converters   - Annotations (@Entity, @Dao, @Query)       |
|   - Compile-Time SQL Verification   - LiveData / Cursor Adapters  |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                        SQLITE ENGINE (C/C++)                      |
|   - B-Tree Storage    - Query Optimizer    - Journal / WAL Disk   |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|            LINUX FILE SYSTEM (/data/data/.../databases/)          |
+-------------------------------------------------------------------+
```

---

### Physical Room Implementation in FemCare

#### 1. Entity: `UserEntity.java`
```java
package com.miniflo.femcare.data;

import androidx.room.Entity;
import androidx.room.PrimaryKey;
import androidx.annotation.NonNull;

@Entity(tableName = "user_table")
public class UserEntity {
    @PrimaryKey
    @NonNull
    public String email; // Primary key matches Firebase authentication identity

    public String name;
    public int age;
    public int averageCycleLength;
    public int periodDuration;
    public boolean isRegular;
    public boolean hasPCOS;
    public boolean hasThyroid;
    public boolean onBirthControl;
    public boolean isPregnant;
    public boolean tryingToConceive;
    public int stressLevel;
    public int sleepHours;
    public int exerciseFrequency;
    public double weightKg;
    public double heightCm;
    public double bmi;
    public String symptoms;

    public UserEntity(@NonNull String email) {
        this.email = email;
    }
}
```

#### 2. Entity: `CycleEntity.java`
```java
package com.miniflo.femcare.data;

import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity(tableName = "cycle_table")
public class CycleEntity {
    @PrimaryKey(autoGenerate = true)
    public int cycleId; // Room autoincrements: 1, 2, 3...

    public long startDateMillis;
    public long endDateMillis;
    public int cycleLength;
    public int periodDuration;
    public boolean isIrregular;

    public CycleEntity(long startDateMillis, long endDateMillis, int cycleLength, int periodDuration) {
        this.startDateMillis = startDateMillis;
        this.endDateMillis = endDateMillis;
        this.cycleLength = cycleLength;
        this.periodDuration = periodDuration;
        this.isIrregular = cycleLength > 45; // Auto-flagging irregular cycles
    }
}
```

#### 3. DAO: `UserDao.java`
```java
package com.miniflo.femcare.data;

import androidx.lifecycle.LiveData;
import androidx.room.Dao;
import androidx.room.Insert;
import androidx.room.OnConflictStrategy;
import androidx.room.Query;

@Dao
public interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertUser(UserEntity user);

    @Query("SELECT * FROM user_table WHERE email = :email LIMIT 1")
    LiveData<UserEntity> getUserByEmail(String email);

    @Query("SELECT COUNT(*) FROM user_table")
    int getUserCountForWarmup();

    @Query("UPDATE user_table SET age = :newAge WHERE email = :userEmail")
    void updateUserAge(String userEmail, int newAge);

    @Query("UPDATE user_table SET averageCycleLength = :cycleLength WHERE email = :userEmail")
    void updateCycleLength(String userEmail, int cycleLength);
}
```

#### 4. DAO: `CycleDao.java`
```java
package com.miniflo.femcare.data;

import androidx.lifecycle.LiveData;
import androidx.room.Dao;
import androidx.room.Insert;
import androidx.room.Query;
import java.util.List;

@Dao
public interface CycleDao {
    @Insert
    void insertCycle(CycleEntity cycle);

    @Query("SELECT * FROM cycle_table ORDER BY startDateMillis DESC")
    LiveData<List<CycleEntity>> getAllCycles();

    @Query("SELECT * FROM cycle_table ORDER BY startDateMillis DESC LIMIT 6")
    List<CycleEntity> getRecentCyclesForMath();

    @Query("DELETE FROM cycle_table WHERE cycleId = :cycleId")
    void deleteCycle(int cycleId);
}
```

#### 5. Database Singleton: `AppDatabase.java`
```java
package com.miniflo.femcare.data;

import android.content.Context;
import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

@Database(entities = {UserEntity.class, CycleEntity.class}, version = 1, exportSchema = false)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
    public abstract CycleDao cycleDao();

    private static volatile AppDatabase INSTANCE;

    public static AppDatabase getInstance(final Context context) {
        if (INSTANCE == null) {
            synchronized (AppDatabase.class) { // Thread-safe Double-Checked Locking
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
                                    AppDatabase.class, "femcare_database")
                            .fallbackToDestructiveMigration()
                            .build();
                }
            }
        }
        return INSTANCE;
    }
}
```

---

### Local Storage Comparisons

| Metric | Room Persistence Library | Raw SQLite (`SQLiteOpenHelper`) | SharedPreferences |
| :--- | :--- | :--- | :--- |
| **SQL Verification** | Compile-time checking prevents invalid syntax | Runtime execution; syntax errors cause crashes | No SQL support (flat XML file) |
| **Data Structure** | Relational tables, columns, indexes | Relational tables, columns, indexes | Key-value pairs (`String`, `boolean`, `int`) |
| **Threading** | Prevents main thread queries by default | Allows queries on main thread (causes ANR) | Reads/writes on main thread or background |
| **Complex Queries**| Supports complex JOINs, aggregations, LIMIT | Supports complex JOINs, aggregations, LIMIT | None; requires loading entire file into RAM |
| **Observable Data**| Native integration with `LiveData`, `Flow`, `RxJava`| Requires manual `ContentObserver` callbacks | `OnSharedPreferenceChangeListener` |
| **Ideal Use Case** | Structured relational entities (`User`, `Cycle`) | Legacy projects not using Android Jetpack | Lightweight settings (`pref_dark_mode`, flags) |

---

# PART 7 — OFFLINE-FIRST ARCHITECTURE AND SYNCHRONISATION

### 1. Conceptual Framework of Offline-First Architecture

An "Offline-First" application treats local persistence as the **single source of truth for the presentation layer**. Rather than treating the absence of internet as an exceptional error state, offline-first design assumes network connectivity is inherently unstable, high-latency, and intermittent.

#### Architectural Principles:
1. **Immediate Local Mutation:** All write operations (logging a symptom, modifying cycle dates) are written directly to the local database first.
2. **Instant UI Response:** The UI updates instantaneously based on the local write, never blocking user interaction with full-screen network progress spinners.
3. **Background Synchronization Queue:** Mutations are appended to an outbound sync queue. A background synchronization worker consumes this queue and dispatches payloads to the cloud server when connectivity is established.
4. **Eventual Consistency:** Local and cloud databases may temporarily diverge in state, but given time and network availability, both nodes will reach identical states.

---

### 2. Conflict Resolution Strategies

When multiple writes occur to the same entity (e.g., a user modifies a note on their tablet while offline and edits the same note on their phone), conflict resolution determines which mutation survives:

1. **Last-Write-Wins (LWW) via Epoch Timestamps (FemCare's Model):**
   - Every mutation carries a high-precision UTC millisecond timestamp (`System.currentTimeMillis()`).
   - When synchronizing, if a cloud record has `timestamp = 1730260000` and the incoming local record has `timestamp = 1730260500`, the local record overwrites the cloud because its timestamp is greater.
2. **Field-Level Merging (Three-Way Merge):**
   - Instead of overwriting an entire document, mutations only update dirty fields using `SetOptions.merge()`.
   - If Device A updates `waterCount` and Device B updates `symptoms`, Firestore merges both without conflict.
3. **Idempotency & Duplicate Elimination:**
   - To avoid inserting duplicate records when a network request is retried after a drop, each record must carry a deterministic client-generated UUID or date key (e.g., `daily_logs/2026-10-30`). Even if the sync worker uploads the payload three times, the Firestore document ID ensures it is written to the same document key rather than creating duplicates.

---

### 3. Complete 7-Step Synchronization Flow

```
[User logs symptom]
        |
        v
1. Local Database Write (Room / SharedPreferences)
        |
        v
2. Mark Sync Status: "PENDING_SYNC"
        |
        v
3. Inspect Network Connectivity State
       /      \
[Online]      [Offline]
   |             |
   |             +--> 4. WorkManager enqueues OneTimeWorkRequest with Constraint:
   |                     NetworkType.CONNECTED
   |                         |
   |                         v (Wait for device to regain Wi-Fi / LTE)
   +-------------------------+
   |
   v
5. Sync Worker executes: Reads all PENDING_SYNC records
   |
   v
6. Dispatches batch payload to Cloud Firestore
   |
   +---> [Success] ---> Update local record status to "SYNCED"
   |
   +---> [Failure / Token Error] ---> Apply Exponential Backoff retry policy
   |
   v
7. Cloud Firestore Snapshot Listener updates local cache (Eventual Consistency achieved)
```

---

### Actual Implementation vs. Proposed Reference Implementation

#### A. What Actually Exists in the FemCare Codebase:
- **Profile Synchronization (`LoginActivity.java`):** On sign-in, `syncFirestoreToRoom` pulls the cloud document and inserts it into Room SQLite (`user_table`).
- **Cloud-First Save Contract (`TrackFragment.java`, `CalendarFragment.java`):** Daily symptoms and cycle edits are dispatched directly to Cloud Firestore. Upon cloud success, a local in-app notification is pushed to `LocalNotificationStore`.
- **Notification Offline Cache (`LocalNotificationStore.java`):** Completely offline-first. Stores notification feed items as a JSON array in `SharedPreferences`. Reads and writes operate locally first; `NotificationsFragment` merges local and cloud feeds.
- **Degradation Detection (`FirebaseAuthState.java`):** Detects token refresh errors and defers background sync tasks by 2 minutes to prevent battery drain.

#### B. Proposed Production-Grade Implementation:
To achieve a fully offline-first architecture across all fragments:
1. Introduce a `syncStatus` column in Room:
   ```sql
   ALTER TABLE cycle_table ADD COLUMN syncStatus TEXT DEFAULT 'SYNCED'; -- PENDING_INSERT, PENDING_UPDATE, SYNCED
   ```
2. When the user taps "Save" in `TrackFragment`, save to Room first with `PENDING_INSERT`.
3. Dispatch a `OneTimeWorkRequest` constrained to `NetworkType.CONNECTED`.
4. The worker transmits the record to Firestore, receives server confirmation, and flips `syncStatus` to `SYNCED`.

---

# PART 8 — CYCLE PREDICTION LOGIC

### 1. Menstrual Cycle Biology & Technical Modeling

The menstrual cycle is modeled computationally using four chronological metrics:
1. **Cycle Start Date ($T_{start}$):** The first day of noticeable menstrual bleeding (Day 1).
2. **Period Duration ($D_{period}$):** The consecutive days of active bleeding (typically 3 to 7 days).
3. **Cycle Length ($L_{cycle}$):** The total number of days between the first day of one period and the first day of the subsequent period (clinical normal: 21 to 35 days; statistical mode: 28 days).
4. **Luteal Phase Duration ($D_{luteal}$):** The time elapsed between ovulation and the next cycle start. Biologically, the luteal phase is remarkably constant at approximately 14 days.

---

### 2. Core Mathematical Formulas

#### Next Estimated Period Start:
$$\text{NextPeriodMillis} = \text{LastPeriodStartMillis} + (L_{cycle} \times 86,400,000\text{ ms})$$

#### Ovulation Day:
$$\text{OvulationMillis} = \text{NextPeriodMillis} - (14 \times 86,400,000\text{ ms})$$

#### Fertile Window:
$$\text{FertileWindowStart} = \text{OvulationMillis} - (5 \times 86,400,000\text{ ms})$$
$$\text{FertileWindowEnd} = \text{OvulationMillis} + (1 \times 86,400,000\text{ ms})$$

*Explanation:* Human spermatozoa can survive in fertile cervical mucus for up to 5 days, while an unfertilized ovum survives for approximately 24 hours post-ovulation. Thus, the biological fertile window spans 6 to 7 days.

---

### 3. Worked Numerical Example

#### Given User Metrics:
- **Last Recorded Period Start Date:** October 1, 2026 ($T_0$)
- **Average Cycle Length ($L_{cycle}$):** 28 days
- **Period Duration ($D_{period}$):** 5 days
- **Current Observation Date:** October 12, 2026

#### Calculations:
1. **Active Period Window:**
   $$\text{October 1 to October 5, 2026}$$
2. **Next Period Start Date:**
   $$\text{October 1} + 28\text{ days} = \text{October 29, 2026}$$
3. **Ovulation Day:**
   $$\text{October 29} - 14\text{ days} = \text{October 15, 2026}$$
4. **Fertile Window:**
   $$\text{Start} = \text{October 15} - 5\text{ days} = \text{October 10, 2026}$$
   $$\text{End} = \text{October 15} + 1\text{ day} = \text{October 16, 2026}$$
5. **State on October 12:**
   October 12 falls inside $[\text{October 10, October 16}]$. FemCare displays: **"High Chance (Fertile Window)"** in high-contrast pink (`#C2185B`).

---

### 4. The Roll-Forward While-Loop Algorithm in `TodayFragment.java`

A critical bug in basic menstrual trackers occurs when a user does not open the app for several months: the predicted "next period" displays a date in the past! 

FemCare fixes this in `TodayFragment.java` via the **Roll-Forward Algorithm**:

```java
// Exact logic from TodayFragment.java: runPredictionAlgorithm()
long nowMillis = Calendar.getInstance().getTimeInMillis();
long cycleLengthMillis = cycleDays * ONE_DAY_MILLIS;

long activeCycleStartMillis = originalLastPeriodMillis;
long nextPeriodMillis = activeCycleStartMillis + cycleLengthMillis;

// --- THE ROLL-FORWARD FIX ---
// If the user missed logging for months, roll the clock forward 
// cycle-by-cycle until we find the CURRENT or NEXT FUTURE cycle!
while (nextPeriodMillis < nowMillis) {
    activeCycleStartMillis += cycleLengthMillis;
    nextPeriodMillis = activeCycleStartMillis + cycleLengthMillis;
}

// Next period is now guaranteed to be in the present or future
Calendar nextPeriod = Calendar.getInstance();
nextPeriod.setTimeInMillis(nextPeriodMillis);

// Calculate ovulation and fertile window from the active rolled cycle
long ovulationMillis = nextPeriodMillis - (14 * ONE_DAY_MILLIS);
long fertileWindowStart = ovulationMillis - (5 * ONE_DAY_MILLIS);
long fertileWindowEnd = ovulationMillis + (1 * ONE_DAY_MILLIS);
```

#### Progress Percentage Calculation:
To drive the custom `HexagonProgressBar`, the fractional progress through the current cycle is computed:
$$\text{Progress} = \frac{\text{nowMillis} - \text{activeCycleStartMillis}}{\text{nextPeriodMillis} - \text{activeCycleStartMillis}}$$
Clamped between $0.0$ and $1.0$.

---

### 5. Why the App Cannot Guarantee Exact Dates
- **Hypothalamic-Pituitary-Ovarian (HPO) Axis Sensitivity:** Acute psychological stress elevates cortisol and Corticotropin-Releasing Hormone (CRH), which directly suppresses Gonadotropin-Releasing Hormone (GnRH) pulsatility, delaying ovulation.
- **Anovulatory Cycles:** Especially in adolescents and individuals with PCOS, a cycle may occur without an egg being released, making cycle lengths completely unpredictable.
- **No Direct Biomarkers:** FemCare relies entirely on historical date mathematics. Without hardware integration (such as continuous skin temperature sensors or urinary hormone test strips), all predictions remain statistical approximations.

---

# PART 9 — SYMPTOM LOGGING AND CALENDAR

### 1. Symptom Logging Architecture (`TrackFragment.java`)

Symptom logging transforms subjective physical sensations into structured clinical data points.

#### Chip Categories & Selections:
- **Physical Symptoms:** Cramps, Headache, Acne, Backache, Nausea, Bloating.
- **Moods:** Happy, Sad, Anxious, Irritable, Mood Swings.
- **Vaginal Discharge:** Dry, Sticky, Creamy, Watery, Egg White.
- **Energy Levels:** High, Medium, Low, Exhausted.
- **Dynamic "Other" Dialog:** Users can tap an "+ Other" chip which raises a `MaterialAlertDialogBuilder` prompting for custom symptom strings, dynamically appending them to the selection list.
- **Hydration Tracker:** Increment (`+`) and decrement (`-`) buttons maintaining an integer `waterGlasses` counter.

#### Date Key Partitioning:
Symptoms are partitioned strictly by calendar date string: `yyyy-MM-dd` (e.g., `2026-10-30`). 
- This enforces an idempotent 1-to-1 mapping: only one primary log document exists per user per calendar day.
- If a user opens `TrackFragment` on a day they previously logged, the fragment pulls the document and re-toggles all active chips so the user can edit their log rather than creating duplicate entries.

---

### 2. Custom 7-Column Calendar Grid (`CalendarFragment.java`)

`CalendarFragment` implements an interactive monthly calendar without using bloated third-party calendar libraries.

```
+-------------------------------------------------------------------+
|  < [Prev]               October 2026                     [Next] > |
+-------------------------------------------------------------------+
|   Sun   |   Mon   |   Tue   |   Wed   |   Thu   |   Fri   |   Sat   |
+---------+---------+---------+---------+---------+---------+---------+
|         |         |         |         |  1 (P)  |  2 (P)  |  3 (P)  |
+---------+---------+---------+---------+---------+---------+---------+
|  4 (P)  |  5 (P)  |    6    |    7    |    8    |    9    | 10 (F)  |
+---------+---------+---------+---------+---------+---------+---------+
| 11 (F)  | 12 (F)  | 13 (F)  | 14 (F)  | 15 (O)  | 16 (F)  |   17    |
+---------+---------+---------+---------+---------+---------+---------+
|   18    |   19    |   20    |   21    |   22    |   23    |   24    |
+---------+---------+---------+---------+---------+---------+---------+
|   25    |   26    |   27    |   28    | 29 (E)  | 30 (E)  | 31 (E)  |
+-------------------------------------------------------------------+
 Legend: (P) Active Period  (F) Fertile Window  (O) Ovulation  (E) Expected Period
```

#### Calendar Mathematics (`CalendarAdapter`):
For every cell rendered in the 7-column `GridLayoutManager`:
```java
// Calculate the day offset between this cell's date and the user's last period
long cellMillis = cellDate.getTimeInMillis();
int daysDiff = (int) ((cellMillis - lastPeriodMillis) / (24L * 60L * 60L * 1000L));

// Compute modular cycle position (handling negative values for past cycles)
int cycleDay = daysDiff % cycleLength;
if (cycleDay < 0) {
    cycleDay = (cycleLength + cycleDay) % cycleLength;
}

// 1. Period Bleeding Phase
if (cycleDay >= 0 && cycleDay < periodDuration) {
    if (daysDiff >= 0 && daysDiff < cycleLength) {
        holder.cellDayText.setBackground(createCircleBg("#C2185B", true)); // Solid Crimson
        dayStatus = "Period Day " + (cycleDay + 1);
    } else {
        holder.cellDayText.setBackground(createCircleBg("#C2185B", false)); // Hollow Outline
        dayStatus = "Expected Period";
    }
}
// 2. Ovulation Day
else if (cycleDay == cycleLength - 14) {
    holder.cellDayText.setBackground(createCircleBg("#F44336", true)); // Solid Red
    dayStatus = "Ovulation Day";
}
// 3. Fertile Window (5 days prior to ovulation through 1 day prior)
else if (cycleDay >= cycleLength - 19 && cycleDay <= cycleLength - 13) {
    holder.cellDayText.setBackground(createCircleBg("#F8BBD0", true)); // Soft Pink Halo
    dayStatus = "Fertile Window";
}
```

#### Gesture Detection:
`CalendarFragment` registers a `GestureDetector.SimpleOnGestureListener`. When horizontal fling velocity exceeds 100 px/s with a displacement greater than 100 px:
- Swipe Left $\rightarrow$ `changeMonth(1)` (Next Month)
- Swipe Right $\rightarrow$ `changeMonth(-1)` (Previous Month)

---

### 3. MPAndroidChart Trend Visualization (`TrackFragment.java`)

In `fetchSymptomTrends()`, FemCare queries the past 30 days of daily log entries from Cloud Firestore:
1. Iterates through documents in `users/{email}/daily_logs`.
2. Extracts the `symptoms` string array from each document.
3. Maps dates to an $X$-axis index and symptom counts to $Y$-values: `new Entry(xIndex, symptomList.size())`.
4. Renders a cubic smoothed `LineChart` using `LineDataSet`:
   - Line Color: Crimson (`#C2185B`).
   - Fill: Gradient under the curve.
   - $X$-Axis: Custom `IndexAxisValueFormatter` displaying date strings (`MM/dd`).


---

# PART 10 — WORKMANAGER AND NOTIFICATIONS

### 1. 12 Essential Background Processing Questions Answered

#### 1. What is background processing?
Background processing refers to executing application logic off the main UI thread, typically when the application is not actively visible in the foreground (e.g., app is backgrounded, user is using another app, screen is locked, or app process is killed).

#### 2. What is WorkManager?
WorkManager is part of Android Jetpack and the recommended solution for **deferrable, guaranteed background work**. It chooses the optimal execution path (JobScheduler on API 23+, or a combination of AlarmManager and BroadcastReceivers on older APIs) while respecting OS battery-saving mechanisms like Doze mode.

#### 3. Why is it useful?
- **Guaranteed Execution:** Even if the user kills the app or restarts their device, WorkManager persists task definitions in an internal SQLite database and resumes execution when system constraints are met.
- **Battery Optimization:** Batches background execution, preventing apps from continually waking the CPU.
- **Constraint Handling:** Delays execution until defined hardware conditions (e.g., network connectivity, charging state) are satisfied.

#### 4. What is a Worker?
An abstract class (`androidx.work.Worker`) that defines the actual unit of work to be performed synchronously on a background thread in its `doWork()` method.

#### 5. What is a WorkRequest?
The base class representing a request to execute a `Worker`. It encapsulates the worker class, execution schedule, tags, and system constraints.

#### 6. What is a OneTimeWorkRequest?
A `WorkRequest` designed to execute exactly once. Used in FemCare for immediate synchronization tasks triggered by user events (e.g., logging a symptom or logging in).

#### 7. What is a PeriodicWorkRequest?
A `WorkRequest` designed to repeat on a cyclical interval. The minimum allowable interval in Android is **15 minutes**. FemCare uses 6-hour, 24-hour, and 7-day periodic requests.

#### 8. What are constraints?
Logical prerequisites that must be satisfied before WorkManager allows a task to run (`Constraints.Builder()`). 
- *FemCare Constraints:* `NetworkType.CONNECTED` (requires active Wi-Fi or cellular data) and `setRequiresBatteryNotLow(true)`.

#### 9. What happens after an app restart or device reboot?
Because WorkManager stores all active work requests in an internal SQLite database (`androidx.work.workdatabase`), the operating system automatically re-registers scheduled alarms and job intents via `ACTION_BOOT_COMPLETED`. When the phone boots up, FemCare's periodic reminders continue running without the user needing to launch the app!

#### 10. What notification permissions are required?
Starting in **Android 13 (API level 33 - Tiramisu)**, displaying notifications requires the runtime permission `android.permission.POST_NOTIFICATIONS`. If denied, background tasks will execute successfully, but `notificationManager.notify()` calls are silently suppressed by the OS.

#### 11. Why might a reminder not appear?
1. The user denied the `POST_NOTIFICATIONS` runtime permission.
2. The OEM manufacturer (Xiaomi, Huawei, Samsung) killed the background process via aggressive custom battery managers ("Auto-Start" disabled).
3. The device entered Doze Mode, delaying deferrable periodic work until the next maintenance window.
4. The user has no network connection, and the work request enforces `NetworkType.CONNECTED`.

#### 12. What is the difference between WorkManager and AlarmManager?
- **WorkManager:** Designed for **deferrable, opportunistic work**. It does *not* guarantee exact-to-the-second execution. It respects Doze mode and waits for optimal system conditions.
- **AlarmManager:** Designed for **exact-time firing** (e.g., a morning alarm clock or a calendar event at 9:00:00 AM sharp). It wakes the CPU aggressively via `setExactAndAllowWhileIdle()`, which consumes significant battery.
- *FemCare's Choice:* WorkManager is used for health reminders because period predictions are date-level events where a 15-minute execution window is acceptable and battery conservation is prioritized.

---

### 2. WorkManager Architecture in FemCare

The background system in FemCare is centralized across two primary classes: `BackgroundTaskScheduler` and `ScheduledNotificationWorker`.

```
[ Application / Activity Triggers ]
  - FemCareApplication.onCreate()
  - DashboardActivity.onCreate()
  - LoginActivity.routeAfterSignIn()
  - TrackFragment (Save daily log)
  - CalendarFragment (Update cycle)
                 |
                 v
+--------------------------------------------------------------------+
|                      BackgroundTaskScheduler                       |
|                                                                    |
|  - scheduleAll(Context):                                           |
|      * work_daily_log_reminder : Periodic (24h), Aligned 20:00    |
|      * work_cycle_alerts       : Periodic (6h)                     |
|      * work_weekly_wellness    : Periodic (7d), Aligned Mon 09:00  |
|                                                                    |
|  - enqueueImmediateSync(Context, reason):                          |
|      * work_immediate_notification_sync : OneTimeWorkRequest       |
|      * Debounced to 20 seconds; deferred if auth error active      |
+--------------------------------------------------------------------+
                                 |
                                 v
+--------------------------------------------------------------------+
|                     ScheduledNotificationWorker                    |
|                                                                    |
|  doWork() executes based on input "mode":                          |
|    - MODE_DAILY     -> Check if today's log exists; quote; BMI     |
|    - MODE_CYCLE     -> Evaluate cycle day; Period in 2d; Ovulation |
|    - MODE_WEEKLY    -> Weekly motivation; BMI nutrition tips       |
|    - MODE_IMMEDIATE -> Runs Daily + Cycle + Weekly in one pass     |
|                                                                    |
|  Under MODE_IMMEDIATE:                                             |
|    - Acquires PowerManager.PARTIAL_WAKE_LOCK                       |
|    - Promotes to Foreground Service via setForegroundAsync()       |
+--------------------------------------------------------------------+
                                 |
                                 v
+--------------------------------------------------------------------+
|                        NotificationPublisher                       |
|                                                                    |
|  1. Writes notification document to Firestore:                     |
|     users/{email}/notifications/{notificationId}                   |
|  2. Upserts to LocalNotificationStore (Offline JSON cache)         |
|  3. Raises system notification via NotificationHelper              |
+--------------------------------------------------------------------+
```

---

### 3. Worker Execution Modes & Business Logic

#### `MODE_DAILY` (Executed every 24 hours, aligned to 8:00 PM):
- Checks Firestore to determine if the user has logged symptoms for today (`users/{email}/daily_logs/{yyyy-MM-dd}`).
- If no document exists and `pref_alert_log` is enabled in `SharedPreferences`, dispatches:
  - *Title:* "Daily Log Reminder"
  - *Body:* *"Don't forget to log your symptoms today to keep your predictions accurate!"*

#### `MODE_CYCLE` (Executed every 6 hours):
- Retrieves `lastPeriodStartMillis` and `averageCycleLength` from the user's Firestore profile.
- Computes `dayInCycle = (todayMidnight - lastPeriodMidnight) / 86400000 % cycleLength`.
- **Period Soon Alert:** If `dayInCycle == cycleLength - 2` (2 days before period onset):
  - *Title:* "Period Reminder"
  - *Body:* *"Your next period is expected in about 2 days. Stay prepared."*
- **Period Today Alert:** If `dayInCycle == 0`:
  - *Title:* "Cycle Update"
  - *Body:* *"Your expected period window starts today."*
- **Fertile Window Alert:** If `dayInCycle == cycleLength - 19`:
  - *Title:* "Fertile Window"
  - *Body:* *"Your fertile window likely starts today."*
- **Ovulation Day Alert:** If `dayInCycle == cycleLength - 14`:
  - *Title:* "Ovulation Day"
  - *Body:* *"Today is likely your ovulation day."*

#### `MODE_IMMEDIATE` (Triggered on login, dashboard launch, or log saves):
- Executes all three checks (Daily, Cycle, Weekly) in an immediate batch.
- **WakeLock Protection:** Acquires a `PowerManager.PARTIAL_WAKE_LOCK` for up to 10 minutes to prevent CPU sleep during network requests.
- **Foreground Service Promotion:** Invokes `setForegroundAsync(createForegroundInfo(context))` to provide high-priority execution immunity from OS process termination.

---

### 4. Legacy Worker: `PeriodReminderWorker.java`

In addition to `ScheduledNotificationWorker`, the codebase contains `PeriodReminderWorker.java`.
- **Code Evidence:** It queries Room SQLite directly:
  ```java
  AppDatabase db = AppDatabase.getInstance(getApplicationContext());
  List<CycleEntity> cycles = db.cycleDao().getRecentCyclesForMath();
  ```
- It inspects the most recent `CycleEntity`, calculates if tomorrow is the start date, and raises an alert.
- **Scheduler Note:** In `BackgroundTaskScheduler.java`, `workManager.cancelUniqueWork("daily_period_reminder")` cancels this legacy worker in favor of the unified `ScheduledNotificationWorker`.

---

# PART 11 — NODE.JS AND EXPRESS BACKEND

### 1. Backend Architecture Fundamentals

In the `backend/` directory of the repository sits a modular Node.js / Express backend service:
- **Node.js:** An open-source, cross-platform JavaScript runtime environment executing code outside a web browser, built on Chrome's V8 engine with an asynchronous event-driven I/O model.
- **Express.js:** A minimalist, unopinionated web framework for Node.js providing robust routing, HTTP utility methods, and middleware infrastructure.
- **Middleware:** Functions that have access to the request object (`req`), response object (`res`), and the `next` function in the application’s request-response cycle. In `server.js`:
  - `cors()`: Enables Cross-Origin Resource Sharing.
  - `express.json()`: Parses incoming HTTP requests with JSON payloads.

---

### 2. Comprehensive Backend Endpoints Master Table

| Route / Endpoint | HTTP Method | Controller Function | Purpose & Functionality | Input Payload / Params | Output Response | Evidence in Repository |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/` | `GET` | Inline in `server.js` | Server health / welcome root | None | `{ "message": "Welcome to FemCare API" }` | `backend/server.js:28` |
| `/health` | `GET` | `healthController.checkHealth` | Operational uptime check | None | `{ "success": true, "message": "API is healthy", "timestamp": "..." }` | `backend/controllers/healthController.js` |
| `/api/logs` | `POST` | `logController.createLog` | Creates a new symptom log entry | `{ "title": "Cramps", "description": "Severe", "severity": 4 }` | `{ "success": true, "message": "...", "data": { "id": "...", ... } }` | `backend/controllers/logController.js:3` |
| `/api/logs` | `GET` | `logController.getAllLogs` | Fetches all stored symptom logs | None | `{ "success": true, "count": N, "data": [...] }` | `backend/controllers/logController.js:19` |
| `/api/logs/:id` | `GET` | `logController.getLogById` | Fetches a single log by ID | Route param: `:id` | `{ "success": true, "data": { ... } }` | `backend/controllers/logController.js:28` |
| `/api/logs/:id` | `PUT` | `logController.updateLog` | Updates an existing log | Route param: `:id`, Body: updates | `{ "success": true, "message": "...", "data": { ... } }` | `backend/controllers/logController.js:40` |
| `/api/logs/:id` | `DELETE`| `logController.deleteLog` | Deletes a log entry | Route param: `:id` | `{ "success": true, "message": "Log deleted successfully" }` | `backend/controllers/logController.js:58` |
| `/api/tips` | `GET` | `tipController.getAllTips` | Returns health & wellness tips | None | `{ "success": true, "data": [ { "id": 1, "title": "Stay Hydrated" }, ... ] }` | `backend/controllers/tipController.js` |
| `/api/posts` | `POST` | `postController.createPost` | Community post creation | `{ "title": "...", "description": "...", "category": "..." }` | `{ "success": true, "data": { "id": "...", ... } }` | `backend/controllers/postController.js:3` |
| `/api/posts` | `GET` | `postController.getAllPosts`| Retrieves all community posts | None | `{ "success": true, "count": N, "data": [...] }` | `backend/controllers/postController.js:29` |
| `/api/posts/:id` | `GET` | `postController.getPostById` | Retrieves a single post | Route param: `:id` | `{ "success": true, "data": { ... } }` | `backend/controllers/postController.js:38` |
| `/api/posts/:id` | `PUT` | `postController.updatePost` | Updates a community post | Route param: `:id`, Body: updates | `{ "success": true, "data": { ... } }` | `backend/controllers/postController.js:44` |
| `/api/posts/:id` | `DELETE`| `postController.deletePost` | Removes a community post | Route param: `:id` | `{ "success": true, "message": "..." }` | `backend/controllers/postController.js:52` |
| `/api/requests`| `POST` | `requestController.createRequest`| Creates a doctor consultation request | `{ "name": "Jane", "message": "Irregular bleeding" }` | `{ "success": true, "data": { "status": "Pending", ... } }` | `backend/controllers/requestController.js` |
| `/api/requests`| `GET` | `requestController.getAllRequests`| Retrieves consultation requests | None | `{ "success": true, "data": [...] }` | `backend/controllers/requestController.js:17` |
| `/api/saved` | `POST` | `savedController.savePost` | Bookmarks a community post | `{ "postId": "abc123" }` | `{ "success": true, "data": { "id": "...", ... } }` | `backend/controllers/savedController.js` |
| `/api/saved` | `GET` | `savedController.getSavedPosts`| Retrieves bookmarked posts | None | `{ "success": true, "data": [...] }` | `backend/controllers/savedController.js:27` |

---

### 3. Repository Reality: In-Memory Arrays vs. Resume Claim

> **CRITICAL INTERVIEW FACT:** In `logController.js`, `postController.js`, `requestController.js`, and `savedController.js`, data is stored in plain JavaScript arrays:
> ```javascript
> let logs = []; // In-memory database array
> let posts = [];
> let requests = [];
> let savedPosts = [];
> ```
> **This means:**
> 1. If the Node.js server process restarts, **all stored records are immediately lost**.
> 2. **The Android app does NOT currently invoke these endpoints.** The Android app communicates directly with Cloud Firestore and Firebase Auth, and calls OSM/Google APIs via Volley.
> 3. Retrofit is declared in `app/build.gradle.kts`, but no Retrofit API client has been instantiated in Java.

#### Truthful Spoken Explanation for Candidate:
> *"In our architecture plan, we designed a Node.js/Express microservice backend to handle community posts, consultation requests, and shared health tips. I built the Express routing structure with full CRUD controllers using in-memory arrays for rapid prototyping and imported Retrofit in Gradle. However, for our core client release, we prioritized direct Firebase SDK integration for real-time offline sync. The Node.js service is currently decoupled and serves as the architectural foundation for our upcoming phase-two backend migration."*

---

### 4. Backend Architectural Comparisons

| Dimension | Direct Firebase SDK (FemCare Client) | Custom Node.js / Express Backend |
| :--- | :--- | :--- |
| **Network Architecture** | Direct TLS websocket / HTTP2 stream from Android to GCP | Android connects to REST API, which queries DB |
| **Development Speed** | Extremely fast; zero server infrastructure to maintain | Slower; requires writing routes, auth middleware, DB pooling |
| **Offline Synchronization** | Handled natively by Firebase Client SDK cache | Must be manually implemented on Android via Room |
| **Complex Business Logic** | Limited to client device or Firebase Cloud Functions | Complete freedom to execute complex Java/Node pipelines |
| **Vendor Lock-In** | High lock-in to Google Cloud Platform | Zero lock-in; deployable on AWS, Docker, Linux VM |
| **Cost at Scale** | Expensive on heavy read/write volumes | Predictable, fixed server compute costs |

---

# PART 12 — SOURCE CODE WALKTHROUGH

### 1. Master File-by-File Reference Table

| Filename | Class / Module | Layer | Primary Responsibility | Important Methods | Data Handled | Dependencies |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `FemCareApplication.java` | `FemCareApplication` | App Core | Global initialization, DB warm-up, global uncaught crash handler | `onCreate()`, `retrieveAndStoreFCMToken()` | Crash logs, FCM tokens | `WorkManager`, `Room`, `FCM` |
| `MainActivity.java` | `MainActivity` | UI / Splash | Splash timer, permission checks, session routing | `onCreate()`, `navigateToNext()` | Auth status, onboarding flags | `FirebaseAuth`, `SharedPreferences` |
| `LoginActivity.java` | `LoginActivity` | UI / Auth | Email & Google login, syncs user profile to Room | `signInUser()`, `syncFirestoreToRoom()` | User credentials, profile doc | `FirebaseAuth`, `GoogleSignIn`, `Room` |
| `RegisterActivity.java` | `RegisterActivity` | UI / Auth | Creates user in Firebase, initializes Firestore profile | `registerUser()` | Email, password, name | `AuthViewModel`, `FirebaseAuth` |
| `BirthDateActivity.java` | `BirthDateActivity` | Onboarding | Captures birth date, calculates age | `calculateAge()`, `saveAgeAndProceed()` | Birth timestamp, calculated age | `AuthViewModel` |
| `UserInfoActivity.java` | `UserInfoActivity` | Onboarding | Height, weight, BMI calculation, regularity | `calculateBMI()`, `saveUserInfo()` | Height, weight, BMI, stress | `AuthViewModel` |
| `TypicalCycleActivity.java`| `TypicalCycleActivity`| Onboarding | Captures baseline cycle length & period duration | `saveCycleDetails()` | Cycle length, period duration | `AuthViewModel` |
| `LastPeriodActivity.java` | `LastPeriodActivity` | Onboarding | Captures last period start and end dates | `savePeriodData()` | Start millis, end millis | `AuthViewModel` |
| `ReproductiveProblemsActivity.java`| `ReproductiveProblemsActivity`| Onboarding | Captures PCOS, thyroid history | `saveReproductiveHealth()` | Boolean flags (PCOS, thyroid) | `AuthViewModel` |
| `LifestyleActivity.java` | `LifestyleActivity` | Onboarding | Sleep hours, exercise frequency, pregnancy | `saveLifestyleData()` | Sleep, exercise, pregnancy | `AuthViewModel` |
| `UneasinessActivity.java` | `UneasinessActivity` | Onboarding | Common uneasiness selection, finalizes onboarding | `finalizeOnboarding()` | Symptoms string list | `AuthViewModel` |
| `DashboardActivity.java` | `DashboardActivity` | UI Shell | BottomNav container, swaps fragments, FAB handler | `onCreate()`, `onNewIntent()` | Navigation state, notification flags | `BottomNavigationView`, `Fragments`|
| `TodayFragment.java` | `TodayFragment` | UI / Hub | Landing screen, roll-forward prediction, hex progress | `fetchDataAndCalculate()`, `runPredictionAlgorithm()` | Cycle math, pregnancy status | `Firestore`, `HexagonProgressBar` |
| `CalendarFragment.java` | `CalendarFragment` | UI / Calendar | 7-column grid, phase styling, swipe gestures, notes | `buildCalendar()`, `openNoteBottomSheet()` | Daily notes, cycle phase dates | `GridLayoutManager`, `Firestore` |
| `TrackFragment.java` | `TrackFragment` | UI / Logging | Chip logging, water counter, 30-day trends chart | `saveDailyLog()`, `fetchSymptomTrends()` | Symptoms, moods, water count | `MPAndroidChart`, `Firestore` |
| `NotificationsFragment.java`| `NotificationsFragment`| UI / Notifs | Merged notification feed (cloud + local cache) | `loadNotifications()`, `markAllAsRead()` | Notification items | `Firestore`, `LocalNotificationStore`|
| `SettingsFragment.java` | `SettingsFragment` | UI / Settings | Reminders, dark mode, profile edit, PDF export | `createDetailedPdfReport()`, `shareFile()` | User profile, generated PDF | `PdfDocument`, `FileProvider` |
| `MedicalReportActivity.java`| `MedicalReportActivity`| UI / Documents| MediaStore camera & picker, local report storage | `openCamera()`, `openFilePicker()`, `saveReport()` | Images, PDFs, report metadata | `FileProvider`, `Glide` |
| `FindDoctorActivity.java` | `FindDoctorActivity` | UI / Location | OSM MapView, Overpass API & Google Places query | `fetchLocationAndClinics()`, `queryOverpassApi()` | Clinic coordinates, ratings | `osmdroid`, `Volley`, `FusedLocation`|
| `ClinicAdapter.java` | `ClinicAdapter` | UI / Adapter | Binds clinic list with distance and ratings | `onBindViewHolder()` | List of `Clinic` objects | `RecyclerView.Adapter` |
| `HexagonProgressBar.java` | `HexagonProgressBar`| Custom View | Draws geometric cycle progress hexagon on canvas | `onDraw()`, `setProgress()` | Normalized progress (0.0 to 1.0) | `Canvas`, `Paint`, `Path` |
| `AppDatabase.java` | `AppDatabase` | Data / Room | Room database singleton (`femcare_database`) | `getInstance()`, `userDao()`, `cycleDao()` | SQLite tables | `RoomDatabase` |
| `UserDao.java` | `UserDao` | Data / Room | User table SQL operations | `insertUser()`, `getUserByEmail()` | `UserEntity` rows | `Room` |
| `CycleDao.java` | `CycleDao` | Data / Room | Cycle table SQL operations (recent 6 limit) | `insertCycle()`, `getRecentCyclesForMath()` | `CycleEntity` rows | `Room` |
| `UserEntity.java` | `UserEntity` | Data / Room | Schema model for `user_table` | Constructor | User profile fields | `@Entity` |
| `CycleEntity.java` | `CycleEntity` | Data / Room | Schema model for `cycle_table` | Constructor | Cycle length, dates, irregular flag | `@Entity` |
| `AuthRepository.java` | `AuthRepository` | Data Broker | Dual-write broker coordinating Firestore and Room | `registerUser()`, `updateUserInfo()` | User profile and cycle records | `FirebaseAuth`, `Firestore`, `Room`|
| `AuthViewModel.java` | `AuthViewModel` | ViewModel | Exposes LiveData and delegates to AuthRepository | `register()`, `updateAge()`, `updatePeriodData()`| Primitive types, LiveData | `AndroidViewModel` |
| `BackgroundTaskScheduler.java`| `BackgroundTaskScheduler`| Background | Central WorkManager scheduler (24h, 6h, 7d, sync) | `scheduleAll()`, `enqueueImmediateSync()` | WorkManager requests | `WorkManager` |
| `ScheduledNotificationWorker.java`| `ScheduledNotificationWorker`| Background | Execution engine for daily, cycle, and weekly checks | `doWork()`, `runCycleAlertChecks()` | Firestore documents | `Worker`, `WakeLock` |
| `DailyHealthWorker.java` | `DailyHealthWorker` | Background | Evaluates missed logs and 2-day period warnings | `doWork()` | Daily log docs | `Worker`, `Tasks.await()` |
| `PeriodReminderWorker.java`| `PeriodReminderWorker`| Background | Legacy worker querying Room for 1-day period alert | `doWork()` | `CycleEntity` via `CycleDao` | `Worker`, `AppDatabase` |
| `NotificationPublisher.java`| `NotificationPublisher`| Notification | Writes Firestore notifs, local cache, system alerts | `publishAsync()`, `publishSync()` | Notification payload | `Firestore`, `LocalNotificationStore`|
| `NotificationHelper.java` | `NotificationHelper` | Notification | Creates NotificationChannel, builds system status bar | `showNotification()` | System notification specs | `NotificationCompat` |
| `LocalNotificationStore.java`| `LocalNotificationStore`| Cache | Offline-first JSON notification feed in Prefs | `upsert()`, `getAll()`, `markRead()` | JSON array of notifications | `SharedPreferences`, `JSONArray` |
| `FirebaseAuthState.java` | `FirebaseAuthState` | Resilience | Token error detection, backoff timer, dialogs | `isAuthTokenError()`, `shouldDeferBackgroundWork()` | Exception patterns | `SharedPreferences` |
| `MyFirebaseMessagingService.java`| `MyFirebaseMessagingService`| Push FCM | Handles FCM token refresh and incoming cloud messages | `onNewToken()`, `onMessageReceived()` | FCM tokens, RemoteMessage | `FirebaseMessagingService` |

---

### 2. Top 15 Most Important Files & Functions

1. **`TodayFragment.java` $\rightarrow$ `runPredictionAlgorithm()`:** The mathematical heartbeat of the application, computing period arrival, roll-forward offsets, fertile window, and ovulation.
2. **`CalendarFragment.java` $\rightarrow$ `CalendarAdapter.onBindViewHolder()`:** Custom arithmetic mapping date cells to period, fertile, and ovulation color styles.
3. **`TrackFragment.java` $\rightarrow$ `persistDailyLog()`:** Multi-symptom packaging, Firestore document persistence, timeout protection, and background sync triggering.
4. **`BackgroundTaskScheduler.java` $\rightarrow$ `scheduleAll()`:** Configures periodic WorkManager jobs with network constraints and daily/weekly delays.
5. **`ScheduledNotificationWorker.java` $\rightarrow$ `doWork()`:** Multi-mode execution engine handling WakeLocks, Foreground Service promotion, and Firestore alert evaluation.
6. **`AuthRepository.java` $\rightarrow$ `registerUser()` & `updatePeriodData()`:** Dual-layer broker writing to Cloud Firestore and Room SQLite simultaneously.
7. **`AppDatabase.java` $\rightarrow$ `getInstance()`:** Thread-safe singleton providing access to Room DAOs and enforcing destructive migration fallback.
8. **`CycleDao.java` $\rightarrow$ `getRecentCyclesForMath()`:** Room SQL query pulling the last 6 cycles for weighted average calculations.
9. **`SettingsFragment.java` $\rightarrow$ `createDetailedPdfReport()`:** Native Android `PdfDocument` generation drawing medical reports, watermarks, and symptom tables.
10. **`NotificationPublisher.java` $\rightarrow$ `publishSync()`:** Multi-destination alert dispatcher writing to Firestore, local JSON cache, and system status bar.
11. **`LocalNotificationStore.java` $\rightarrow$ `upsert()`:** Offline-first caching engine serializing notification feed items into SharedPreferences JSON.
12. **`FirebaseAuthState.java` $\rightarrow$ `isAuthTokenError()`:** Resilience detector inspecting exception chains to prevent background retry storms during token degradation.
13. **`FindDoctorActivity.java` $\rightarrow$ `queryOverpassApi()`:** REST networking via Volley querying OpenStreetMap Overpass servers for gynecological clinics.
14. **`MedicalReportActivity.java` $\rightarrow$ `saveReport()`:** Streams camera capture or picked documents into app external storage with MIME-type resolution.
15. **`FemCareApplication.java` $\rightarrow$ `onCreate()`:** Initializes background schedulers, executes Room warmup query, and registers the global uncaught crash handler.

---

### 3. Detailed Execution Paths for Core Flows

#### 1. Registration Flow (`VERIFIED IMPLEMENTED`):
1. User enters email, password, and name in `RegisterActivity.java`.
2. Taps "Register" $\rightarrow$ triggers `AuthViewModel.register(email, password, name)`.
3. `AuthRepository.registerUser()` calls `firebaseAuth.createUserWithEmailAndPassword(email, password)`.
4. On success, constructs `UserEntity` and writes a baseline profile map to Firestore: `users/{email}`.
5. In an `ExecutorService` background thread, executes `userDao.insertUser(newUser)`.
6. LiveData posts `true` $\rightarrow$ `RegisterActivity` launches `BirthDateActivity` to initiate onboarding.

#### 2. Login Flow (`VERIFIED IMPLEMENTED`):
1. User enters credentials in `LoginActivity.java` (or uses Google Sign-In).
2. `firebaseAuth.signInWithEmailAndPassword(email, password)` executes.
3. On success, `routeAfterSignIn()` pulls document `users/{email}` from Firestore.
4. `syncFirestoreToRoom()` extracts fields in a background thread and calls `userDao.insertUser(entity)`.
5. Checks `onboardingComplete` boolean flag:
   - If `true` $\rightarrow$ invokes `BackgroundTaskScheduler.scheduleAll(this)` and routes to `DashboardActivity`.
   - If `false` $\rightarrow$ routes to `BirthDateActivity`.

#### 3. Adding a Cycle Record (`VERIFIED IMPLEMENTED`):
1. In `LastPeriodActivity.java` (or edit dialog in `CalendarFragment.java`), user selects period start and end dates.
2. Calls `AuthViewModel.updatePeriodData(email, duration, startMillis, listener)`.
3. `AuthRepository` updates Firestore document `users/{email}` with `lastPeriodStartMillis`, `lastPeriodEndMillis`, and `periodDuration`.
4. In background thread, inserts `CycleEntity` into Room: `cycleDao.insertCycle(new CycleEntity(...))`.
5. Triggers `BackgroundTaskScheduler.enqueueImmediateSync(context, "period_data_saved")`.

#### 4. Adding a Symptom Log (`VERIFIED IMPLEMENTED`):
1. User opens `TrackFragment.java` via the Dashboard FAB.
2. Selects chips (e.g., "Cramps", "Bloating"), adjusts water glasses to 8.
3. Taps "Save Daily Log" $\rightarrow$ invokes `saveDailyLog(view)`.
4. Wrapped in `wrapCloudSaveWithTimeout` (15s safety net).
5. Dispatches payload to Firestore: `users/{email}/daily_logs/{yyyy-MM-dd}` via `SetOptions.merge()`.
6. On Firestore acknowledgment:
   - Local in-app notification cached in `LocalNotificationStore`.
   - WorkManager immediate sync enqueued (`daily_log_saved`).
   - Line chart (`fetchSymptomTrends`) and calendar grid re-queried and refreshed.

#### 5. Viewing Calendar Information (`VERIFIED IMPLEMENTED`):
1. User selects "Calendar" tab in `DashboardActivity`.
2. `CalendarFragment.onCreateView()` initializes a 7-column `GridLayoutManager`.
3. `fetchUserData()` retrieves cycle length, duration, and last period timestamp from Firestore.
4. `buildCalendar()` populates date cells for the current month.
5. `CalendarAdapter.onBindViewHolder()` calculates `daysDiff` and `cycleDay` for each date, styling background circles (Crimson, Red, or Pink halo).
6. Tapping a date opens `openNoteBottomSheet()` to view or add notes in subcollection `users/{email}/notes/{dateKey}`.

#### 6. Reminder Scheduling Flow (`VERIFIED IMPLEMENTED`):
1. `FemCareApplication.onCreate()` or `DashboardActivity.onCreate()` invokes `BackgroundTaskScheduler.scheduleAll(context)`.
2. Cancels any legacy `daily_period_reminder` work.
3. Builds three `PeriodicWorkRequest` instances with `NetworkType.CONNECTED` constraints:
   - Daily log reminder: 24h interval, delayed to 8:00 PM.
   - Cycle alerts: 6h interval.
   - Weekly wellness: 7d interval, delayed to Monday 9:00 AM.
4. Enqueues them via `workManager.enqueueUniquePeriodicWork()` using `ExistingPeriodicWorkPolicy.KEEP` (prevents resetting schedules).

---

# PART 13 — DATABASE DESIGN

### 1. Data Schema & Relationships

```
+-----------------------------------+             +----------------------------------+
|            USER_TABLE             |             |           CYCLE_TABLE            |
|       (Room Local Database)       |             |      (Room Local Database)       |
+-----------------------------------+             +----------------------------------+
| PK  email               : TEXT    | 1         * | PK  cycleId          : INTEGER   |
|     name                : TEXT    |-------------|     startDateMillis  : INTEGER   |
|     age                 : INTEGER |             |     endDateMillis    : INTEGER   |
|     averageCycleLength  : INTEGER |             |     cycleLength      : INTEGER   |
|     periodDuration      : INTEGER |             |     periodDuration   : INTEGER   |
|     isRegular           : BOOLEAN |             |     isIrregular      : BOOLEAN   |
|     hasPCOS             : BOOLEAN |             +----------------------------------+
|     hasThyroid          : BOOLEAN |
|     onBirthControl      : BOOLEAN |             +----------------------------------+
|     isPregnant          : BOOLEAN |             |       DAILY_LOGS (Subcollection) |
|     tryingToConceive    : BOOLEAN |             |      (Cloud Firestore NoSQL)     |
|     stressLevel         : INTEGER |             +----------------------------------+
|     sleepHours          : INTEGER | 1         * | PK  dateKey          : STRING    |
|     exerciseFrequency   : INTEGER |-------------|     timestamp        : INTEGER   |
|     weightKg            : REAL    |             |     waterCount       : INTEGER   |
|     heightCm            : REAL    |             |     periodEnds       : BOOLEAN   |
|     bmi                 : REAL    |             |     symptoms         : ARRAY     |
|     symptoms            : TEXT    |             +----------------------------------+
+-----------------------------------+
```

---

### 2. SQL Schemas for Room Tables

#### Table: `user_table`
```sql
CREATE TABLE IF NOT EXISTS user_table (
    email TEXT PRIMARY KEY NOT NULL,
    name TEXT,
    age INTEGER NOT NULL,
    averageCycleLength INTEGER NOT NULL,
    periodDuration INTEGER NOT NULL,
    isRegular INTEGER NOT NULL,
    hasPCOS INTEGER NOT NULL,
    hasThyroid INTEGER NOT NULL,
    onBirthControl INTEGER NOT NULL,
    isPregnant INTEGER NOT NULL,
    tryingToConceive INTEGER NOT NULL,
    stressLevel INTEGER NOT NULL,
    sleepHours INTEGER NOT NULL,
    exerciseFrequency INTEGER NOT NULL,
    weightKg REAL NOT NULL,
    heightCm REAL NOT NULL,
    bmi REAL NOT NULL,
    symptoms TEXT
);
```

#### Table: `cycle_table`
```sql
CREATE TABLE IF NOT EXISTS cycle_table (
    cycleId INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
    startDateMillis INTEGER NOT NULL,
    endDateMillis INTEGER NOT NULL,
    cycleLength INTEGER NOT NULL,
    periodDuration INTEGER NOT NULL,
    isIrregular INTEGER NOT NULL
);
```

---

### 3. Schema Evolution & Migration Strategy

In `AppDatabase.java`, the database is defined as:
```java
@Database(entities = {UserEntity.class, CycleEntity.class}, version = 1, exportSchema = false)
```
with `.fallbackToDestructiveMigration()`.
- **The Risk:** If you alter `UserEntity` (e.g., adding a column) and increment version to 2 without writing a migration, Room drops all existing tables and re-creates them, wiping the user's local cycle history!
- **Production Migration Defense:**
  ```java
  static final Migration MIGRATION_1_2 = new Migration(1, 2) {
      @Override
      public void migrate(@NonNull SupportSQLiteDatabase database) {
          database.execSQL("ALTER TABLE user_table ADD COLUMN bloodGroup TEXT DEFAULT 'Unknown'");
      }
  };
  // Registered in Room.databaseBuilder().addMigrations(MIGRATION_1_2)...
  ```

---

# PART 14 — PRIVACY, SECURITY AND RELIABILITY

### 1. Medical Data Sensitivity & Regulatory Frameworks

Menstrual and reproductive health tracking involves **Special Category Personal Data** under the European Union General Data Protection Regulation (GDPR Article 9) and sensitive Protected Health Information (PHI) under the Health Insurance Portability and Accountability Act (HIPAA) principles:
- **Reproductive Privacy Risks:** Post-Roe v. Wade legal developments highlight the risk of subpoenaed cycle tracking data in jurisdictions with anti-abortion statutes.
- **Data Minimization:** FemCare limits data collection to strictly operational parameters. It does not record GPS location in the cycle database (location in `FindDoctorActivity` is ephemeral and never persisted to Firestore).

---

### 2. Security Implementations in FemCare

#### 1. Secure File Sharing via Android FileProvider
In `SettingsFragment.java` and `MedicalReportActivity.java`, medical files (PDFs and images) are shared using `androidx.core.content.FileProvider`:
- **Why not `file://` URIs?** Passing raw file paths (`file:///storage/...`) via an Intent exposes private file system structures and throws a `FileUriExposedException` on Android 7.0+ (API 24+).
- **How FileProvider Works:** It generates a secure `content://` URI (`content://com.miniflo.femcare.fileprovider/...`) mapped through XML configuration (`res/xml/file_paths.xml`).
- **Temporary Scoped Permissions:** The URI is passed with `Intent.FLAG_GRANT_READ_URI_PERMISSION`, granting the receiving app (e.g., Gmail, WhatsApp) read-only access strictly for the duration of the Intent handling task.

#### 2. Cloud Data Isolation
Every cloud document in Firestore resides beneath the authenticated user's email: `users/{email}/*`. As established in Part 5, Firestore Security Rules enforce that requests where `request.auth.token.email != userEmail` are rejected with `PERMISSION_DENIED`.

#### 3. Application Crash Resilience & Self-Healing
In `FemCareApplication.java`, FemCare installs a global `Thread.setDefaultUncaughtExceptionHandler`:
```java
Thread.setDefaultUncaughtExceptionHandler((thread, throwable) -> {
    try {
        Log.e("AppCrash", "Uncaught exception", throwable);
        SharedPreferences prefs = getSharedPreferences("FemCarePrefs", Context.MODE_PRIVATE);
        prefs.edit()
             .putString("last_crash", throwable.toString())
             .putLong("last_crash_ts", System.currentTimeMillis())
             .apply();

        // Schedule graceful restart via AlarmManager
        Intent restartIntent = new Intent(getApplicationContext(), MainActivity.class);
        restartIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        PendingIntent pendingIntent = PendingIntent.getActivity(
                getApplicationContext(), 42, restartIntent, 
                PendingIntent.FLAG_ONE_SHOT | PendingIntent.FLAG_IMMUTABLE);
        
        AlarmManager am = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
        if (am != null) {
            am.set(AlarmManager.RTC, System.currentTimeMillis() + 1000, pendingIntent);
        }
    } finally {
        android.os.Process.killProcess(android.os.Process.myPid());
        System.exit(2);
    }
});
```
- *Interview Defense:* *"If an unhandled NullPointerException occurs in a background thread, rather than showing an abrupt 'App has stopped' dialog to the user, our crash handler writes a diagnostic trace to SharedPreferences and schedules a clean restart through AlarmManager in 1 second."*


---

# PART 15 — TESTING AND LIMITATIONS

### 1. Codebase Testing Reality vs. Recommended Strategy

> **TRUTHFUL REPOSITORY EVIDENCE:**
> If you inspect `app/src/test/java/com/miniflo/femcare/ExampleUnitTest.java` and `app/src/androidTest/java/com/miniflo/femcare/ExampleInstrumentedTest.java`, you will find only the default Android Studio starter templates:
> ```java
> assertEquals(4, 2 + 2); // ExampleUnitTest.java
> assertEquals("com.miniflo.femcare", appContext.getPackageName()); // ExampleInstrumentedTest.java
> ```
> There are **zero custom unit tests or automated UI instrumentation tests committed in the repository**. 
> All validation performed during development was **manual end-to-end testing** on Android Studio emulators (Pixel 7, API 34) and physical Android hardware.

#### Truthful Interview Response:
> *"During our rapid prototyping phase, our testing strategy was focused on rigorous manual functional validation across device emulators and physical hardware. We thoroughly verified edge cases—such as leap years, 60-day delayed app openings, network drops during symptom saves, and permission denials. In our production roadmap, establishing an automated CI/CD testing pipeline using JUnit 4, Mockito, and Room In-Memory Database testing is our immediate priority."*

---

### 2. Comprehensive Recommended Testing Suite

| Test Category | Target Component | Test Scenario / Edge Case | Testing Tool & Method |
| :--- | :--- | :--- | :--- |
| **Unit Test** | `TodayFragment` Prediction Math | Roll-forward math with last period 90 days ago | JUnit 4 test verifying `nextPeriodMillis > nowMillis` |
| **Unit Test** | `CycleEntity` Constructor | Cycle length > 45 days | Asserts that `isIrregular == true` |
| **Unit Test** | `UserInfoActivity` BMI Math | Height 160cm, Weight 55kg | Asserts that calculated BMI equals 21.48 |
| **Database Test**| `CycleDao` & `UserDao` | Insert and retrieve last 6 cycles | `Room.inMemoryDatabaseBuilder()` in AndroidTest |
| **Worker Test** | `ScheduledNotificationWorker` | Verify 24h delay and mode execution | `WorkManagerTestInitHelper` with TestDriver |
| **Integration**| `AuthRepository` | Registration failure handling | Mockito mocking `FirebaseAuth` exceptions |
| **UI Test** | `TrackFragment` | Selecting 3 chips and tapping save | Espresso: `onView(withId(R.id.chipCramps)).perform(click())` |
| **Network Test**| `FirebaseAuthState` | Intercepting `unauthenticated` error | Asserts `shouldDeferBackgroundWork()` returns `true` |

---

### 3. Inherent Limitations of the Application

1. **Prediction Inaccuracies in Irregular Cycles:** The prediction engine assumes mathematical periodicity. In individuals with severe Polycystic Ovary Syndrome (PCOS), perimenopause, or high physiological stress, cycles do not follow a fixed schedule.
2. **Cold-Start Problem:** For brand-new users, predictions rely on a static baseline input (defaulting to 28 days) until multiple cycles are logged to generate a personalized moving average.
3. **Decoupled Node.js Backend:** The Node.js Express server is not connected to the Android client; its controllers rely on non-persistent in-memory arrays.
4. **Android OEM Background Killers:** Aggressive battery management software on devices (Xiaomi MIUI, Huawei EMUI, Samsung OneUI) may suppress WorkManager periodic execution unless the user manually grants "Unrestricted Battery" and "Auto-Start" permissions.
5. **No End-to-End Cryptographic Encryption:** While transport is encrypted via HTTPS/TLS and Firestore rules isolate user data, records are not end-to-end encrypted on device storage using SQLCipher or Android Keystore.

---

# PART 16 — IMPLEMENTATION CHALLENGES AND PERSONAL CONTRIBUTION

### 1. Hardest Technical Challenges & Resolutions

#### Challenge 1: Progressive Firebase Authentication Token Degradation
- **The Problem:** In development builds, after prolonged background execution or network switching, Cloud Firestore write operations would intermittently hang indefinitely or throw `SecureToken` / `unauthenticated` exceptions. The UI buttons remained disabled, freezing the interface.
- **The Resolution:** 
  1. Built `FirebaseAuthState.java` to inspect exception throwables for auth-token error strings.
  2. Implemented a 2-minute backoff window (`shouldDeferBackgroundWork`) preventing WorkManager from hammering Firebase when auth tokens are broken.
  3. Engineered `wrapCloudSaveWithTimeout()` in `TrackFragment` and `CalendarFragment`, introducing a 15-second safety timer that automatically re-enables UI controls and alerts the user if cloud callbacks fail to respond.

#### Challenge 2: Room & Firestore Dual-Persistence Coordination
- **The Problem:** Coordinating simultaneous writes to Cloud Firestore and Room SQLite without blocking the Android Main Thread (causing Application Not Responding - ANR errors).
- **The Resolution:** Designed `AuthRepository` with an `ExecutorService` (`Executors.newSingleThreadExecutor()`). Network operations execute asynchronously via Firebase Tasks, and upon callback, database disk writes are immediately dispatched to the background executor, communicating back to the UI via `Handler(Looper.getMainLooper())` or `MutableLiveData`.

---

### 2. Honest Defense of AI-Assisted Development

Interviewers appreciate honesty, technical ownership, and critical code literacy far more than fabricated claims of writing every boilerplate line from scratch.

#### Ideal Response:
> *"I utilized modern developer productivity tools, including AI coding assistants, to accelerate boilerplate generation—such as XML layout constraints, repetitive DAO interfaces, and Gradle dependency configs. However, I took complete ownership of the system architecture, debugging, and verification.*
>
> *For example, when AI-generated code produced a classic compile error in `LifestyleActivity.java` due to non-final variable captures inside Java lambdas, I diagnosed and fixed the issue by declaring local variables effectively final. Similarly, when Firebase background tasks suffered from silent token failures, I personally architected the `FirebaseAuthState` backoff resilience layer and timeout wrappers to ensure the app never freezes in production.*
>
> *Every single class in this repository was reviewed, modified, compiled, and validated line-by-line by me."*

---

# PART 17 — MINIMUM 80 INTERVIEW QUESTIONS & ANSWERS

---

### LEVEL 1: BEGINNER (Questions 1 to 25)

#### Q1: What is the minimum Android SDK supported by FemCare?
**Answer:** FemCare specifies `minSdk = 24` in `app/build.gradle.kts`. This means the application can run on devices with Android 7.0 (Nougat) and higher, ensuring backward compatibility across ~95% of active Android devices globally.
- *Follow-up 1:* What happens if you try to install this APK on an Android 6.0 (API 23) device?
  - *Answer:* The Google Play Store or Android package installer (`pm`) blocks installation with `INSTALL_FAILED_OLDER_SDK`.
- *Follow-up 2:* Why didn't you set `minSdk = 21`?
  - *Answer:* API 24 introduced the Java 8 Streams API and `java.time` backports natively without requiring heavy desugaring libraries.

#### Q2: What is compileSdk vs. targetSdk?
**Answer:** `compileSdk` (set to 36 in FemCare) is the version of Android APIs that Gradle uses to compile your Java source code into DEX bytecode. `targetSdk` (set to 36) informs the Android operating system that the app has been built and tested against the behavioral rules and security constraints of that API level.

#### Q3: What is an Activity in Android?
**Answer:** An `Activity` represents a single focused screen that provides a user interface for user interaction (e.g., `LoginActivity`). It serves as an entry point for user interaction and manages a dedicated window hierarchy.

#### Q4: What is the Android Activity Lifecycle?
**Answer:** The lifecycle is a state machine managed by the OS: `onCreate()` $\rightarrow$ `onStart()` $\rightarrow$ `onResume()` $\rightarrow$ `onPause()` $\rightarrow$ `onStop()` $\rightarrow$ `onDestroy()`.
- *Follow-up 1:* In which callback should you release heavy resources like location listeners or map views?
  - *Answer:* In `onPause()` or `onStop()`. In `FindDoctorActivity.java`, `mapView.onPause()` and `requestQueue.cancelAll()` are called in `onPause()`.
- *Follow-up 2:* What happens when the device is rotated?
  - *Answer:* By default, the OS destroys and recreates the Activity (`onPause` $\rightarrow$ `onStop` $\rightarrow$ `onDestroy` $\rightarrow$ `onCreate`), resetting non-persistent state unless preserved in a `ViewModel` or `onSaveInstanceState`.

#### Q5: What is a Fragment and why use it?
**Answer:** A `Fragment` is a reusable, modular portion of an Activity’s UI with its own lifecycle. FemCare uses Fragments (`TodayFragment`, `CalendarFragment`, `TrackFragment`, etc.) inside `DashboardActivity` so users can switch tabs smoothly without the performance overhead of launching separate activities.

#### Q6: What is an Intent? Explain Explicit vs. Implicit Intents.
**Answer:** An `Intent` is an asynchronous messaging object used to request an action from another app component.
- **Explicit Intent:** Specifies the exact target class (e.g., `new Intent(MainActivity.this, DashboardActivity.class)`).
- **Implicit Intent:** Declares a general action to be handled by any capable app on the device (e.g., `new Intent(Intent.ACTION_VIEW, Uri.parse("geo:0,0?q=clinic"))` to open Google Maps).

#### Q7: What is a Context in Android?
**Answer:** A `Context` provides access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting intents, and obtaining system services (`ALARM_SERVICE`, `POWER_SERVICE`).
- *Follow-up 1:* What is the difference between Application Context and Activity Context?
  - *Answer:* Application Context lives for the entire lifecycle of the app process and is safe for singletons (`AppDatabase.getInstance(getApplicationContext())`). Activity Context is tied to an individual screen's lifecycle; holding an Activity Context in a static variable causes memory leaks.

#### Q8: What is SharedPreferences?
**Answer:** A lightweight mechanism to store and retrieve persistent key-value pairs of primitive data types (`boolean`, `int`, `long`, `String`) stored in an XML file in private app storage.

#### Q9: What is the difference between `apply()` and `commit()` in SharedPreferences?
**Answer:** `commit()` writes data to disk synchronously and blocks the calling thread, returning a boolean. `apply()` writes to in-memory storage immediately and commits to disk asynchronously in the background, preventing UI thread blockage.

#### Q10: What is a RecyclerView and why is it preferred over ListView?
**Answer:** `RecyclerView` is an advanced and flexible version of `ListView`. It enforces the `ViewHolder` pattern to recycle views as they scroll off-screen, drastically reducing memory allocation and preventing garbage collection stutter.

#### Q11: What is the role of an Adapter in RecyclerView?
**Answer:** An Adapter acts as a bridge between a data source (e.g., `List<Clinic>`) and the `RecyclerView.ViewHolder`. It inflates item XML layouts in `onCreateViewHolder()` and binds data to views in `onBindViewHolder()`.

#### Q12: What is `AndroidManifest.xml`?
**Answer:** The root configuration file of every Android app. It declares application components (Activities, Services, Broadcast Receivers, Content Providers), required hardware features (Camera), and user permissions (`INTERNET`, `POST_NOTIFICATIONS`).

#### Q13: What is Gradle in Android?
**Answer:** Gradle is an advanced build automation tool that compiles source code, merges resources, manages external library dependencies, and packages everything into an installable APK or AAB file.

#### Q14: What is Firebase Authentication?
**Answer:** A cloud identity service providing client-side SDKs and server backends to authenticate users using passwords, phone numbers, or federated identity providers like Google and Facebook.

#### Q15: What is Cloud Firestore?
**Answer:** A cloud-hosted, NoSQL, document-oriented database that allows real-time synchronization, offline caching, and direct client access from mobile devices.

#### Q16: What is a Document vs. a Collection in Firestore?
**Answer:** A **Document** is a set of key-value pairs (similar to a JSON object). A **Collection** is a container for Documents. In FemCare, `users` is a collection; `hiten@example.com` is a document within that collection.

#### Q17: What is the purpose of `View.GONE` vs. `View.INVISIBLE`?
**Answer:** `View.INVISIBLE` hides the view but retains its layout dimensions, occupying space in the layout hierarchy. `View.GONE` hides the view completely and removes it from layout measurement, occupying zero space.

#### Q18: What is the Android Main Thread (UI Thread)?
**Answer:** The primary execution thread in an Android application responsible for dispatching UI events, inflating views, and rendering graphics. Any operation that blocks the Main Thread for more than 5 seconds triggers an **ANR (Application Not Responding)** dialog.

#### Q19: Why can you not execute database queries on the Main Thread?
**Answer:** Disk I/O operations have unpredictable latency. Reading or writing to SQLite on the main thread causes UI frame drops (jank) and can easily trigger an ANR crash.

#### Q20: What is Material Design?
**Answer:** A design system developed by Google that provides pre-built, aesthetically polished UI components (`MaterialButton`, `TextInputEditText`, `BottomNavigationView`) with standard elevation, ripple animations, and accessibility.

#### Q21: What is a FloatingActionButton (FAB)?
**Answer:** A circular button that floats above the interface to promote the primary, most common action on a screen. In FemCare, the FAB on the Dashboard opens `TrackFragment` for immediate daily logging.

#### Q22: What is a BottomSheetDialog?
**Answer:** A modal view that slides up from the bottom of the screen. In `CalendarFragment.java`, FemCare uses it to allow users to add notes for a selected calendar date without navigating away from the calendar.

#### Q23: What is a WakeLock in Android?
**Answer:** A power management mechanism that allows an application to keep the device CPU active even when the screen turns off, preventing background calculations from being frozen by the OS.

#### Q24: What is a Foreground Service?
**Answer:** A service that performs an operation noticeable to the user (requiring a persistent status bar notification). The Android OS grants Foreground Services the highest process priority, ensuring they are not killed when system RAM is low.

#### Q25: What is an APK?
**Answer:** Android Package Kit—the compiled file format used by Android to distribute and install mobile applications. It contains compiled `.dex` bytecode, resources, assets, certificates, and the manifest.

---

### LEVEL 2: INTERMEDIATE (Questions 26 to 55)

#### Q26: What is the Room Persistence Library?
**Answer:** An architectural component of Android Jetpack that provides an object-relational mapping abstraction over SQLite, offering compile-time SQL verification, observable data types, and reduced boilerplate code.

#### Q27: How does Room prevent memory leaks in database instances?
**Answer:** By implementing the **Singleton Pattern** with double-checked locking in `AppDatabase.getInstance(context)`. It also requires passing `context.getApplicationContext()` so the database does not hold a reference to an Activity Context.

#### Q28: What is the purpose of `@PrimaryKey(autoGenerate = true)`?
**Answer:** It instructs SQLite to automatically assign an autoincrementing integer identifier ($1, 2, 3\dots$) to every newly inserted row in the table (used in `CycleEntity.java`).

#### Q29: What is LiveData in Android?
**Answer:** An observable data holder class that is lifecycle-aware. It only updates active observers (Activities or Fragments) that are in an active lifecycle state (`STARTED` or `RESUMED`), preventing memory leaks and crashes due to stopped activities.

#### Q30: What is ViewModel in Android Architecture Components?
**Answer:** A class designed to store and manage UI-related data in a lifecycle-conscious way. It survives configuration changes (such as screen rotations), ensuring data does not need to be re-fetched when an Activity is recreated.

#### Q31: How does `AuthViewModel` communicate with `AuthRepository` in FemCare?
**Answer:** `AuthViewModel` holds an instance of `AuthRepository`. When an Activity calls `viewModel.register()`, the ViewModel delegates the operation to `repository.registerUser()`, returning a `LiveData<Boolean>` that the Activity observes.

#### Q32: What is WorkManager and how does it guarantee task execution?
**Answer:** WorkManager persists task metadata in an internal SQLite database. When the device restarts or recovers network connectivity, WorkManager automatically reschedules and executes the tasks according to defined constraints.

#### Q33: What is `ExistingPeriodicWorkPolicy.KEEP`?
**Answer:** A policy passed to `enqueueUniquePeriodicWork()` that instructs WorkManager to retain the existing scheduled periodic task if one already exists with the same unique name, preventing duplicate scheduled jobs.

#### Q34: What is the minimum interval for a `PeriodicWorkRequest`?
**Answer:** 15 minutes. Attempting to schedule a periodic work request with an interval less than 15 minutes causes WorkManager to automatically clamp the interval to 15 minutes to preserve device battery.

#### Q35: What is `NotificationChannel` and why is it mandatory?
**Answer:** Introduced in Android 8.0 (API 26), notification channels allow developers to categorize notifications. Users can configure permissions, sounds, and vibration for each channel individually in system settings. Posting a notification without a valid channel on API 26+ causes it to be discarded.

#### Q36: How does `NotificationHelper` handle Android 13 notification permissions?
**Answer:** Before calling `NotificationManager.notify()`, it checks:
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU &&
    ContextCompat.checkSelfPermission(context, Manifest.permission.POST_NOTIFICATIONS) != PERMISSION_GRANTED) {
    return;
}
```

#### Q37: What is `FileProvider` in Android?
**Answer:** A specialized subclass of `ContentProvider` that facilitates secure sharing of files associated with an app by creating a `content://` URI instead of a `file://` URI, preventing `FileUriExposedException`.

#### Q38: How does FemCare generate a PDF medical report?
**Answer:** In `SettingsFragment.java`, it utilizes Android’s native `android.graphics.pdf.PdfDocument`. It creates a page, accesses its `Canvas`, draws text and tables using `Paint`, writes the document to a file in `getExternalFilesDir()`, and shares it via `FileProvider`.

#### Q39: What is Volley and where is it used in FemCare?
**Answer:** Volley is an HTTP networking library developed by Google that manages asynchronous network requests and request queues. FemCare uses Volley in `FindDoctorActivity.java` to dispatch a `JsonObjectRequest` to the OpenStreetMap Overpass API.

#### Q40: What is the OpenStreetMap Overpass API?
**Answer:** A read-only API that serves custom selected parts of the OpenStreetMap data across the world. FemCare uses it to query medical facilities (`amenity=clinic`, `amenity=doctors`, `amenity=hospital`) within a 10 km radius of the user's coordinates.

#### Q41: How does `FindDoctorActivity` handle clinic discovery if the Google Maps API key is missing?
**Answer:** It uses a **Hybrid Model**: it fetches and displays free clinic coordinates from OpenStreetMap via `osmdroid` and launches navigation using an implicit Google Maps intent, while gracefully disabling Google rating filter chips.

#### Q42: What is the difference between `set()` and `update()` in Cloud Firestore?
**Answer:** `update()` modifies only the specified fields of an existing document; if the document does not exist, the operation fails. `set(data, SetOptions.merge())` creates the document if it does not exist, or merges the specified fields into the existing document.

#### Q43: How does `TrackFragment` prevent duplicate daily logs?
**Answer:** It uses the calendar date string (`yyyy-MM-dd`) as the Firestore document ID (`users/{email}/daily_logs/{dateKey}`). Any subsequent save on the same day overwrites or merges into that exact document.

#### Q44: What is the purpose of `Tasks.await()`?
**Answer:** A method in Google Play Services Tasks API that blocks the calling thread until an asynchronous task completes, returning the result synchronously. 
- *Follow-up:* Can you call `Tasks.await()` on the Main Thread?
  - *Answer:* No! Calling `Tasks.await()` on the Main Thread throws an `IllegalStateException` because it blocks the UI thread. It is used inside `ScheduledNotificationWorker.doWork()`, which runs on a background thread.

#### Q45: How does FemCare handle dark mode?
**Answer:** It checks a boolean flag in `SharedPreferences` (`pref_dark_mode`). In `MainActivity.java` and `SettingsFragment.java`, it calls:
```java
AppCompatDelegate.setDefaultNightMode(isDarkMode ? MODE_NIGHT_YES : MODE_NIGHT_NO);
```

#### Q46: What is the purpose of `fallbackToDestructiveMigration()` in Room?
**Answer:** It tells Room that if a schema change occurs without a migration path, it is permitted to drop existing tables and re-create them. While useful during development, in production it must be replaced with proper `Migration` classes to avoid data loss.

#### Q47: What is `FusedLocationProviderClient`?
**Answer:** A Google Play Services location API that intelligently manages underlying location technologies (GPS, Wi-Fi, Cellular) to provide the best possible location accuracy while minimizing battery consumption.

#### Q48: How does `FindDoctorActivity` obtain user location?
**Answer:** It calls `fusedLocationClient.getCurrentLocation(Priority.PRIORITY_BALANCED_POWER_ACCURACY, cancellationToken)` to fetch a high-accuracy latitude and longitude point.

#### Q49: What is `Glide` and why is it used in `MedicalReportActivity`?
**Answer:** Glide is an image loading and caching library. FemCare uses it to load picked image URIs and camera thumbnails into `ImageView` instances efficiently, handling automatic downsampling and memory caching.

#### Q50: What is the purpose of `res/xml/file_paths.xml`?
**Answer:** It defines the physical file system directories that `FileProvider` is permitted to expose as secure content URIs. FemCare specifies `<external-files-path name="reports" path="."/>`.

#### Q51: What is the difference between `Serializable` and `Parcelable` in Android?
**Answer:** `Serializable` is a standard Java interface that uses reflection, which is slow and creates many temporary objects. `Parcelable` is an Android-specific interface where developers write explicit serialization code, offering significantly faster inter-process communication.

#### Q52: What is the function of `LocalNotificationStore`?
**Answer:** It provides an offline-first cache of notification items stored as a serialized JSON array in `SharedPreferences`, enabling the notifications tab to display history immediately even when offline.

#### Q53: What is `FirebaseMessagingService`?
**Answer:** A service that extends Firebase Cloud Messaging to receive push notification payloads (`onMessageReceived`) and capture newly generated FCM device registration tokens (`onNewToken`).

#### Q54: What happens if an image picked in `MedicalReportActivity` is very large (e.g., 20 MB)?
**Answer:** The app reads the input stream via `ContentResolver`, copies the bytes into a local file in `FemCareReports`, and uses `resolveDisplaySize()` to format the size in MB. Glide automatically scales down the visual bitmap to prevent OutOfMemory (OOM) errors.

#### Q55: What is `android.hardware.camera` feature in `AndroidManifest.xml`?
**Answer:** It informs the Google Play Store that the app requires a physical camera device to operate.

---

### LEVEL 3: ADVANCED (Questions 56 to 80+)

#### Q56: How does FemCare prevent UI freezing during Firebase token refresh degradation?
**Answer:** It uses `wrapCloudSaveWithTimeout()` in `TrackFragment` and `CalendarFragment`. When a save starts, the button is disabled and a 15-second timer is posted to `Handler(Looper.getMainLooper())`. If Firestore callbacks do not return within 15 seconds, the handler automatically re-enables the button and displays a retry message.

#### Q57: What is the function of `FirebaseAuthState.shouldDeferBackgroundWork()`?
**Answer:** When a Firestore operation throws an authentication or token-related error, `markAuthError()` records the current timestamp in `SharedPreferences`. `shouldDeferBackgroundWork()` checks if less than 2 minutes have elapsed since that error; if so, it suppresses immediate background sync tasks to prevent battery-draining retry loops.

#### Q58: What is an Anovulatory Cycle, and how does it challenge algorithmic prediction?
**Answer:** An anovulatory cycle is a menstrual cycle where the ovaries fail to release an oocyte. Because no corpus luteum forms, progesterone is not produced, causing irregular and prolonged follicular phases. Since FemCare assumes a 14-day luteal phase, its prediction algorithm will fail to accurately forecast the subsequent cycle start.

#### Q59: Explain the mathematical logic behind `HexagonProgressBar`.
**Answer:** It calculates elapsed cycle time:
$$\text{Progress} = \frac{\text{currentTime} - \text{cycleStartTime}}{\text{nextCycleTime} - \text{cycleStartTime}}$$
In `onDraw(Canvas canvas)`, it constructs a closed geometric `Path` using 6 calculated $(X, Y)$ coordinate vertices at $60^\circ$ radial increments ($R \cos\theta, R \sin\theta$), applying a `PathMeasure` trim to draw the progress stroke.

#### Q60: Why does `FemCareApplication` run a database warmup thread on startup?
**Answer:** Room initializes the underlying SQLite database lazily on the first query. To ensure the Android Studio **Database Inspector** connects immediately upon app launch, `FemCareApplication.onCreate()` launches a background thread that executes a lightweight query (`userDao.getUserCountForWarmup()`), forcing immediate database file opening.

#### Q61: What is Double-Checked Locking in Singleton implementation?
**Answer:** A concurrency design pattern that reduces synchronization overhead. In `AppDatabase.getInstance()`:
```java
if (INSTANCE == null) {
    synchronized (AppDatabase.class) {
        if (INSTANCE == null) {
            INSTANCE = Room.databaseBuilder(...).build();
        }
    }
}
```
The first check avoids entering the synchronized block once the instance is created; the second check guarantees that only one instance is created if two threads enter simultaneously.

#### Q62: Why must the singleton variable `INSTANCE` be declared `volatile`?
**Answer:** The `volatile` keyword ensures that multiple threads handle the singleton instance correctly by guaranteeing memory visibility and preventing compiler instruction reordering during object instantiation.

#### Q63: What is the Roll-Forward Algorithm in `TodayFragment`?
**Answer:** If a user’s recorded `lastPeriodStartMillis` occurred months in the past, a static addition of cycle length results in a predicted date that is still in the past. The roll-forward algorithm executes a `while (nextPeriodMillis < nowMillis)` loop, stepping forward by `cycleLengthMillis` until `nextPeriodMillis` is strictly in the future.

#### Q64: How would you prevent a Man-in-the-Middle (MITM) attack on API endpoints?
**Answer:** By implementing **SSL/TLS Certificate Pinning** using OkHttp’s `CertificatePinner`. This ensures the Android app only trusts specific cryptographic public keys, rejecting rogue certificates even if installed in the device trust store.

#### Q65: What is Doze Mode in Android and how does it affect WorkManager?
**Answer:** Introduced in Android 6.0, Doze Mode reduces battery consumption by deferring background CPU and network activity when the device is unplugged, stationary, and screen-off. WorkManager respects Doze mode by deferring tasks until system maintenance windows, unless triggered via high-priority FCM push messages.

#### Q66: What is a Partial WakeLock?
**Answer:** A `PowerManager.PARTIAL_WAKE_LOCK` ensures that the device CPU continues running while the screen and keyboard backlight are turned off. FemCare acquires one in `ScheduledNotificationWorker.MODE_IMMEDIATE` with a 10-minute safety timeout to ensure critical health checks complete.

#### Q67: What happens if a WakeLock is acquired but never released?
**Answer:** It causes a severe **battery drain bug** where the device CPU is prevented from entering deep sleep, draining the user's battery rapidly. FemCare guards against this by always placing `wakeLock.release()` inside a `finally` block and passing a maximum timeout to `acquire(10 * 60 * 1000L)`.

#### Q68: What is the architectural role of `SetOptions.merge()` in Firestore?
**Answer:** When writing to a document, `SetOptions.merge()` updates the fields present in the payload without overwriting the entire document. If omitted, fields not present in the payload are deleted from the document.

#### Q69: How would you architect bi-directional offline synchronization with conflict resolution?
**Answer:** 
1. Maintain a local sync queue table in Room (`sync_queue`).
2. When offline, log mutations with client timestamps and a unique UUID.
3. When network connectivity is regained, dispatch mutations via WorkManager.
4. On the cloud server, use Firestore Transactions to inspect server timestamps. If the server timestamp is newer, reject the mutation; otherwise, commit the write and notify the client.

#### Q70: What is the difference between REST and WebSocket communication?
**Answer:** REST is an architectural style based on stateless HTTP request-response cycles. WebSockets provide a persistent, bi-directional, full-duplex communication channel over a single TCP connection, which Firebase Firestore uses under the hood for real-time document listeners.

#### Q71: How does FemCare's global crash handler recover from unexpected crashes?
**Answer:** In `FemCareApplication.java`, `Thread.setDefaultUncaughtExceptionHandler` intercepts unhandled exceptions, writes the stack trace to `SharedPreferences`, and configures `AlarmManager` to fire an explicit Intent launching `MainActivity` after 1,000 ms before terminating the dying process with `Process.killProcess(Process.myPid())`.

#### Q72: What is ProGuard / R8 and why is it used?
**Answer:** R8 is Android’s code shrinker and obfuscator. It removes unused code and resources, renames classes and fields with short identifier names (e.g., `a.b.c`), and optimizes bytecode to reduce APK size and prevent reverse engineering.

#### Q73: Why is `exportSchema = false` set in `AppDatabase.java`?
**Answer:** It instructs Room not to export the database schema JSON into a compile-time folder. In production, this should be set to `true` to allow version-controlled schema validation and automated migration testing.

#### Q74: What is the danger of storing plain-text medical notes in `SharedPreferences`?
**Answer:** `SharedPreferences` XML files are stored unencrypted in plain text on the device filesystem. On rooted devices or via ADB backup commands, these files can be extracted, exposing sensitive user health data.

#### Q75: How should sensitive local data be secured in Android?
**Answer:** By using the **Android Jetpack Security (EncryptedSharedPreferences)** library and **SQLCipher** for Room, which encrypts database files using AES-256 with cryptographic keys securely stored in the hardware-backed **Android Keystore System**.

#### Q76: What is the difference between compile-time dependency injection (Dagger/Hilt) and manual injection?
**Answer:** Manual injection (used in FemCare) passes dependencies via constructors (`new AuthRepository(application)`). Hilt/Dagger generates boilerplate code at compile time to inject dependencies automatically, improving modularity and testability.

#### Q77: Why did you use `Tasks.await()` instead of async callbacks inside `ScheduledNotificationWorker`?
**Answer:** WorkManager’s `doWork()` method executes on a background thread and expects a synchronous `Result` return (`Result.success()`, `Result.retry()`, `Result.failure()`). Using asynchronous listeners would cause `doWork()` to return immediately before the cloud operation finishes.

#### Q78: What is the purpose of `clipData` in `SettingsFragment.shareFile()`?
**Answer:** When sharing files via `FileProvider` on Android 4.1+, `Intent.setClipData()` grants explicit read URI permissions to the receiving app, preventing security exceptions on newer Android versions.

#### Q79: How does `IndexAxisValueFormatter` work in MPAndroidChart?
**Answer:** It maps numerical float $X$-values on the chart (e.g., $0.0, 1.0, 2.0$) to human-readable strings (e.g., `"10/28"`, `"10/29"`, `"10/30"`) based on an internal string array.

#### Q80: How does `FindDoctorActivity` prevent geocoder thread saturation?
**Answer:** It uses an `ExecutorService` (`Executors.newSingleThreadExecutor()`) to process geocoding requests sequentially in the background, shutting down the executor in `onDestroy()` via `geocoderExecutor.shutdownNow()`.

---

# PART 18 — COMPARISONS AND TRICK QUESTIONS

### 1. Why Java instead of Kotlin?
**Candidate Response:**
> *"I chose Java to build a deep, foundational understanding of the core Android SDK, object-oriented design patterns, threading primitives, and the underlying JVM/DEX bytecode architecture without relying on language-level syntactical sugar. By mastering Java first—including explicit memory management, interfaces, concurrency with Executors, and strict typing—transitioning to Kotlin's coroutines and extension functions becomes straightforward."*

### 2. Why Android Native instead of Flutter or React Native?
**Candidate Response:**
> *"Cross-platform frameworks introduce an abstraction layer that can limit access to cutting-edge platform APIs and increase APK binary size. FemCare relies heavily on deep native Android subsystems: WorkManager’s background scheduling, partial WakeLocks, native Canvas rendering for our HexagonProgressBar, hardware location services via FusedLocationClient, and native PDF generation using android.graphics.pdf.PdfDocument. Building natively ensures zero-overhead performance, battery efficiency, and direct hardware control."*

### 3. Why Firebase instead of a Custom Backend?
**Candidate Response:**
> *"For a client-focused mobile application, Firebase provides an enterprise-grade, managed infrastructure with zero operational maintenance. It offers built-in offline synchronization, real-time Firestore listeners, automated TLS encryption, and secure federated OAuth authentication out of the box. While we designed a modular Node.js/Express backend in the repository for future microservice experimentation, Firebase allowed us to deliver an ultra-responsive, cloud-synced user experience rapidly."*

### 4. Why Room instead of SharedPreferences?
**Candidate Response:**
> *"SharedPreferences is designed strictly for flat, key-value primitive pairs (such as user settings or boolean flags). It is not indexed, does not support SQL querying, and requires loading the entire XML file into RAM. Menstrual tracking requires structured relational data—such as querying the last 6 cycles ordered by date for mathematical moving averages. Room provides a robust relational database with compile-time SQL verification, indexing, and transactional integrity."*

### 5. Why Room instead of direct SQLite (`SQLiteOpenHelper`)?
**Candidate Response:**
> *"Direct SQLite requires writing repetitive boilerplate code, manually managing Cursors, and handling database schema versions without compile-time safety. A misspelled column name in raw SQLite will only crash at runtime in production. Room verifies all SQL queries at compile time, automatically converts database rows into Java POJOs, and integrates natively with lifecycle-aware LiveData."*

### 6. Why both Room AND Firebase?
**Candidate Response:**
> *"They solve two distinct architectural problems:
> - **Room** provides **zero-latency local persistence and offline continuity**. The user can open their health records instantly on an airplane without a network connection.
> - **Firebase** provides **cloud durability, multi-device synchronization, and secure account recovery**. If the user loses their phone, their medical tracking history is safely restored upon login. Together, they create a resilient, hybrid offline-first system."*

### 7. Why Firestore instead of MySQL?
**Candidate Response:**
> *"Firestore is a NoSQL document database that scales horizontally and provides native mobile client SDKs with offline caching and real-time push subscriptions. Traditional relational databases like MySQL require an intermediary REST API server, connection pooling, and manual client-side cache synchronization. Firestore allows direct, secure mobile querying guarded by declarative Security Rules."*

### 8. Why WorkManager instead of AlarmManager?
**Candidate Response:**
> *"AlarmManager is designed for exact-to-the-second execution (like an alarm clock), which aggressively wakes the device CPU and drains battery life. WorkManager is designed for **deferrable, battery-conscious background execution**. It respects Doze mode, allows setting hardware constraints (requiring network connectivity and healthy battery), and automatically handles task persistence across device reboots."*

### 9. What is the difference between Authentication and Authorization?
**Candidate Response:**
> *"Authentication verifies the identity of the user—proving **who you are** (e.g., logging in with email and password to receive a verified Firebase UID). Authorization determines permissions—proving **what you are allowed to access** (e.g., Firestore Security Rules enforcing that user A cannot read or write the symptom logs belonging to user B)."*

### 10. What is the difference between SQL and NoSQL?
**Candidate Response:**
> *"SQL databases (like SQLite/MySQL) are relational, table-based, and enforce strict predefined schemas with ACID transactions and relational JOINs. NoSQL databases (like Cloud Firestore) are non-relational, document-oriented, schema-flexible, and horizontally scalable, optimizing for rapid hierarchical reads and flexible data models."*

### 11. What happens when internet connectivity is lost?
**Candidate Response:**
> *"In FemCare, read operations continue uninterrupted because user profiles and cycle entities are cached locally in Room and SharedPreferences. In our notification module, items are stored in `LocalNotificationStore`. For write operations, our timeout wrapper prevents UI freezing, and future sync tasks are queued until connectivity returns."*

### 12. How do you prevent two users from seeing each other's records?
**Candidate Response:**
> *"Through a combination of client-side path partitioning and server-side Security Rules:
> 1. In client code, all Firestore document paths are namespaced beneath the authenticated user's email: `users/{userEmail}/...`
> 2. In Cloud Firestore Security Rules, we enforce:
>    `allow read, write: if request.auth != null && request.auth.token.email == userEmail;`
> Any unauthorized request is rejected at the database engine level with `PERMISSION_DENIED`."*

### 13. What happens if the prediction is inaccurate?
**Candidate Response:**
> *"Menstrual cycle prediction is a statistical estimation, not a medical diagnosis. If a user’s period arrives earlier or later than predicted, they simply record the actual start date in `LastPeriodActivity` or `CalendarFragment`. The system immediately recalculates average cycle length, shifts future predictions, and updates the roll-forward baseline."*

### 14. What happens when the application is closed or killed?
**Candidate Response:**
> *"The UI components are destroyed by the OS, but scheduled background tasks remain active in WorkManager's persistent database. When periodic intervals trigger (every 6 hours or 24 hours), the Android OS wakes `ScheduledNotificationWorker` in the background to evaluate cycle rules and post system notifications even if the app process has been terminated."*

### 15. How would you scale this application to 1,000,000 users?
**Candidate Response:**
> *"1. Migrate Firestore document keys from plain-text emails to immutable `request.auth.uid` to improve indexing performance.
> 2. Introduce Redis caching and a persistent database (PostgreSQL/MongoDB) in our Node.js backend.
> 3. Implement Cloud Functions for Firebase to offload heavy analytics and notification dispatching from individual devices to serverless cloud workers.
> 4. Enable ProGuard/R8 code shrinking and WebP image compression to minimize bandwidth and app binary size."*

### 16. How would you improve the security of this application?
**Candidate Response:**
> *"1. Implement SQLCipher to encrypt the local Room database (`femcare_database`) with AES-256 encryption.
> 2. Replace standard SharedPreferences with `EncryptedSharedPreferences` backed by the Android Keystore.
> 3. Enforce SSL Certificate Pinning on all outgoing HTTP connections.
> 4. Add biometric authentication (Fingerprint / Face Unlock via Android BiometricPrompt API) before allowing access to sensitive medical logs."*


---

# PART 19 — PROJECT DEFENCE SCRIPTS

### 12 Master Spoken Interview Responses

#### 1. Explain your project.
> *"FemCare is a native Android women's health platform developed in Java. It provides menstrual cycle tracking, algorithmic period and ovulation forecasting, daily multi-symptom logging, and automated background reminders. Architecturally, it utilizes a hybrid data layer combining Google Cloud Firestore for cloud backup and Room SQLite for offline access, alongside an enterprise background scheduling engine powered by Android Jetpack WorkManager."*

#### 2. Why did you select this project?
> *"I wanted to tackle a domain where data privacy, offline reliability, and time-series calculations intersect. Menstrual cycle tracking is not a trivial CRUD application; it requires continuous date arithmetic, background task execution that survives phone reboots, secure handling of personal health records, and a resilient data architecture that remains fully functional when internet access is unavailable."*

#### 3. Why is it useful?
> *"It transforms subjective, scattered daily feelings into structured, clinically valuable medical data. By tracking symptoms alongside cycle phases, users can identify hormonal patterns, prepare for upcoming periods in advance, and compile their health history into a standardized PDF report to share with their gynecologist."*

#### 4. What is its strongest feature?
> *"The automated background scheduling system built with WorkManager. Rather than relying on simple in-app timers that terminate when the app closes, our background engine schedules 24-hour daily logging reminders and 6-hour cycle milestone evaluations. It enforces hardware constraints like network connectivity and battery health, ensuring reliable notification delivery without draining the user's battery."*

#### 5. What is its biggest limitation?
> *"Its reliance on mathematical date modeling rather than direct biological biomarkers. Because the app estimates ovulation mathematically (14 days before the predicted period), it cannot account for sudden hormonal disruptions caused by acute stress, illness, or anovulatory cycles in conditions like PCOS without additional hardware inputs like basal body temperature."*

#### 6. Which part are you most confident about?
> *"The Room SQLite persistence layer and the WorkManager background architecture. I have an in-depth understanding of how Room functions as an abstraction over native SQLite, how DAO interfaces map to compile-time verified queries, and how WorkManager coordinates periodic background tasks while respecting Android Doze mode."*

#### 7. Which part was most challenging?
> *"Handling Firebase authentication token degradation during prolonged offline or background states. When auth tokens expired silently, cloud writes would hang, disabling UI buttons. I diagnosed this and built a dedicated `FirebaseAuthState` resilience handler with backoff timers and safety timeout wrappers to prevent UI freezes."*

#### 8. Which project is your best?
> *"FemCare is my most technically comprehensive project because it demonstrates end-to-end full-stack mobile development: from native UI design and custom Canvas drawing, to relational database modeling with Room, cloud synchronization with Firebase, background scheduling with WorkManager, and RESTful API engineering with Node.js and Express."*

#### 9. How is this better than a simple tracking application?
> *"Most student tracking projects are basic CRUD forms with a local SQLite database and an AlarmManager timer that breaks upon reboot. FemCare implements an enterprise-grade hybrid architecture: real-time cloud synchronization, multi-category symptom logging, a custom 7-column calendar with swipe gestures, on-device native PDF report compilation, and background tasks managed by Jetpack WorkManager that persist across device reboots."*

#### 10. What would you add with another month of development time?
> *"I would prioritize three architectural enhancements:
> 1. Complete the bi-directional offline synchronization queue in Room to support seamless offline write replay with conflict resolution.
> 2. Connect our decoupled Node.js/Express backend with persistent MongoDB/PostgreSQL storage, migrating off client-side Firebase SDK to our own authenticated REST API.
> 3. Implement full automated test coverage using JUnit 4, Mockito, and Espresso."*

#### 11. Would you make it production-ready?
> *"To make FemCare production-ready for the Google Play Store, I would:
> 1. Implement database encryption using SQLCipher for Room and EncryptedSharedPreferences for user settings.
> 2. Replace email-based Firestore document keys with cryptographically secure Firebase UIDs (`users/{uid}`).
> 3. Add biometric authentication (Fingerprint / Face Unlock) to protect sensitive medical logs.
> 4. Establish an automated CI/CD pipeline with strict unit, integration, and security regression testing."*

#### 12. What did you personally do?
> *"I actively reviewed, integrated, tested, and debugged all core modules: designing the Room database schema (`user_table` and `cycle_table`), building the onboarding flow with `AuthViewModel` and `AuthRepository`, engineering the WorkManager background scheduling system, implementing the roll-forward cycle algorithm in `TodayFragment`, and authoring the modular Node.js/Express API."*

---

### How to Respond When an Interviewer Spots an Incomplete Feature

| Interrupted / Challenged On | Terrible Answer (Instant Rejection) | Ideal Professional Answer (Demonstrates Senior Maturity) |
| :--- | :--- | :--- |
| **"Your Node.js backend is not connected to your Android app."** | *"It is connected, maybe you are looking at the wrong branch."* (Dishonest) | *"You are absolutely right. In our current architecture, the Android client communicates directly with Firebase Auth and Firestore for real-time sync. We developed the Node.js/Express server as a standalone modular API to prototype community features and health tips. Connecting the mobile client to our Node backend via Retrofit is our planned phase-two milestone."* |
| **"Your backend uses in-memory arrays, so data resets on restart."** | *"No, it saves to a database somewhere."* (Ignorant) | *"Yes, exactly. The controllers currently use in-memory JavaScript arrays for rapid prototyping during development. In our production roadmap, the next step is replacing those arrays with persistent Mongoose/MongoDB models and adding JWT authentication middleware."* |
| **"You have no unit tests in the repository."** | *"Testing wasn't necessary for this project."* (Unprofessional) | *"That is a fair observation. During our initial development phase, we prioritized rapid feature iteration and manual end-to-end testing across physical devices and emulators. However, for a production healthcare application, automated unit testing with JUnit and Mockito is essential, and establishing a CI/CD test suite is our top priority."* |
| **"Room uses `fallbackToDestructiveMigration()`, which wipes user data on schema changes."** | *"I didn't know that."* (Unprepared) | *"Yes, we enabled `fallbackToDestructiveMigration()` during active schema prototyping to avoid writing migration scripts for every field change. However, for production, I know that destructive migrations are dangerous because they drop tables. I would implement explicit `Migration` objects executing `ALTER TABLE` statements to preserve user data."* |

---

# PART 20 — REALISTIC TECHNICAL INTERVIEW SIMULATION

**Setting:** Technical Interview at a leading IT Services / Product Company.  
**Interviewer:** Senior Principal Architect (Strict, analytical, probing every claim).  
**Candidate:** Hiten Bhurani (Prepared, honest, technically precise).  
**Duration:** ~25 minutes.

---

### Transcript of the Simulation

**Interviewer:**  
*"Good morning, Hiten. I see 'FemCare - Menstrual Tracker' on your resume. You have listed Android Java, Firebase, Room, WorkManager, and Node.js. Can you walk me through the high-level architecture in 60 seconds?"*

**Candidate:**  
*"Good morning! FemCare is a native Android women's health application developed in Java. It features an Activity-Fragment architecture backed by a hybrid persistence model. We use Google Cloud Firestore as our cloud source of truth and Room SQLite for local caching and offline continuity.*  
*On the presentation layer, `DashboardActivity` hosts five modular fragments, including `TodayFragment` which executes our cycle prediction algorithm, and `TrackFragment` which logs multi-category symptoms. Background processing is handled by an enterprise WorkManager engine that schedules 24-hour log reminders and 6-hour cycle milestone checks under strict battery and network constraints. In the repository, we also have a modular Node.js/Express backend engineered for API experimentation."*

---

**Interviewer (Probing Architecture):**  
*"You mentioned Room and Firebase together. Why would you use both? Isn't Cloud Firestore already equipped with offline persistence?"*

**Candidate:**  
*"That’s a great question. While Cloud Firestore does have built-in client caching, relying on it as your sole local database has key limitations:*  
*First, Firestore's cache is managed internally by the SDK—you cannot easily execute complex relational SQL queries, such as our query in `CycleDao` that pulls the last 6 cycles ordered by date to compute a moving average.*  
*Second, Room provides guaranteed, zero-latency local relational access that is completely decoupled from cloud SDK initialization. If there's an authentication token issue with Firebase, our app can still read and write to Room SQLite without interruption."*

---

**Interviewer (Pushing on Implementation):**  
*"Let's look at your Room database. How many tables do you have, and what are their primary keys?"*

**Candidate:**  
*"In `AppDatabase.java`, we define two entities: `UserEntity` and `CycleEntity`. In `UserEntity`, the primary key is the user's `email` as a string, matching our Firebase authentication identifier. In `CycleEntity`, the primary key is an autoincrementing integer `cycleId` generated by Room."*

---

**Interviewer (Challenging Database Design):**  
*"Using an email address as a primary key in a database table? What happens if the user updates their email? Isn't that a violation of relational database normalization?"*

**Candidate:**  
*"You make a valid architectural point. Using a mutable string like an email address as a primary key can create issues if an email changes, and string comparisons are slower than integer or UUID lookups. In our current implementation, we mapped it to email because our Firestore document paths were structured as `users/{email}`. However, in a refactored production schema, the superior approach is to use the immutable, cryptographically generated Firebase `UID` as the primary key (`users/{uid}`), treating email merely as an unindexed attribute."*

---

**Interviewer (Probing Predictions & Math):**  
*"Explain your prediction algorithm. How does the app calculate ovulation and the fertile window?"*

**Candidate:**  
*"The prediction engine in `TodayFragment.java` relies on biological date arithmetic. Given the user's last recorded period start date and their average cycle length, we calculate the next period start date by adding `cycleLength * 86400000 ms`.*  
*Because the biological luteal phase is consistently around 14 days, ovulation is calculated as 14 days before the next period: `NextPeriod - 14 days`. The fertile window spans 5 days prior to ovulation through 1 day after ovulation, reflecting sperm viability of 5 days and ovum lifespan of 24 hours.*  
*Additionally, we implemented a 'roll-forward' while-loop: if a user opens the app after 3 months, the algorithm rolls the cycle forward cycle-by-cycle until it locates the current or future cycle window, preventing the UI from showing past dates."*

---

**Interviewer (Challenging Biological Assumptions):**  
*"What if a user has PCOS and their cycles vary between 28 and 60 days? Your math will fail completely, won't it?"*

**Candidate:**  
*"Yes, that is an inherent limitation of mathematical calendar models. When a user has PCOS, follicular phases fluctuate unpredictably, meaning ovulation is not locked to Day 14. We address this transparently: in `CycleEntity.java`, any cycle exceeding 45 days is automatically flagged as `isIrregular = true`. We explicitly inform the user that calendar predictions are statistical estimates, not clinical diagnoses, and advise consulting a physician when cycle variance exceeds normal bounds."*

---

**Interviewer (Drilling into Background Tasks):**  
*"You have listed WorkManager on your resume. Why didn't you just use `AlarmManager` with a `BroadcastReceiver` to show period reminders?"*

**Candidate:**  
*"AlarmManager is designed for exact-time execution—like an alarm clock waking up at 7:00:00 AM sharp. To do that, AlarmManager must wake the CPU aggressively, which bypasses Android's battery-saving Doze mode and drains battery.*  
*Health reminders are date-level events where a 15-minute or 1-hour execution window is completely acceptable. WorkManager is the Jetpack standard for deferrable, guaranteed work. It respects Doze mode, allows us to set hardware constraints like `NetworkType.CONNECTED` and `BatteryNotLow`, and automatically handles task persistence across device reboots via its internal SQLite database."*

---

**Interviewer (Checking Code Details):**  
*"In `ScheduledNotificationWorker.java`, you used `PowerManager.PARTIAL_WAKE_LOCK`. Why do you need a WakeLock if you're already using WorkManager?"*

**Candidate:**  
*"In `ScheduledNotificationWorker`, when executing in `MODE_IMMEDIATE`—which runs when the user logs in or updates health data—we perform asynchronous network requests to Cloud Firestore using `Tasks.await()`. We acquire a `PARTIAL_WAKE_LOCK` with a 10-minute timeout to guarantee that the device CPU does not enter deep sleep while network packets are being exchanged, and we promote the task using `setForegroundAsync()`. Most importantly, we always release the WakeLock in a `finally` block to prevent battery drain bugs."*

---

**Interviewer (Spotting Incomplete Resume Claims):**  
*"Now, let's talk about your backend. Your resume states: 'Backend API integration using Node.js and Express.' But when I look at your Android code, you're using Firebase SDK and Volley, not calling your Express backend. Why is that?"*

**Candidate (Honest & Strong Response):**  
*"That is accurate. In our repository, we have a fully structured Node.js/Express service in the `backend/` directory with CRUD controllers for logs, posts, and consultation requests. We developed this backend to explore building our own microservice APIs and imported Retrofit in Gradle to prepare for integration.*  
*However, for our initial client release, we focused on direct Firebase integration for real-time offline sync. The Node.js backend currently uses in-memory arrays and operates as a standalone prototype. Connecting the Android client to our custom backend via Retrofit is our planned phase-two architectural migration."*

---

**Interviewer (Testing Security Awareness):**  
*"If an attacker decompiles your APK, what prevents them from wiping your entire Cloud Firestore database?"*

**Candidate:**  
*"Client-side code should never be trusted. The only barrier protecting Firestore is server-side **Firestore Security Rules**. In our rules configuration, we mandate that every incoming request must be authenticated via `request.auth != null`, and that the user's authenticated token email or UID must match the document path: `request.auth.token.email == userEmail`. Any attempt by an attacker to write to another user's document path is blocked at the database engine level with a `403 PERMISSION_DENIED`."*

---

**Interviewer (Closing Evaluation):**  
*"Very good, Hiten. You demonstrated clear architectural understanding, honesty about your codebase, and solid knowledge of Android fundamentals. Thank you."*

---

# PART 21 — LAST-DAY REVISION SHEET

### 1. 60-Second Final Pitch
*"FemCare is a native Android women's health application built in Java. It features a hybrid persistence architecture combining Google Cloud Firestore for real-time cloud synchronization and Room SQLite for zero-latency offline access. It incorporates an automated background execution engine using Android Jetpack WorkManager to calculate cycle milestones, period alerts, and daily reminders while respecting device battery constraints. The app also includes clinical PDF generation via Android's native PdfDocument API and an interactive clinic locator powered by OpenStreetMap and Google APIs."*

---

### 2. Compact ASCII Architecture Blueprint

```
[ UI Layer: 13 Activities + 5 Fragments ]
    |
    v
[ ViewModel & Repository Broker: AuthViewModel -> AuthRepository ]
    |
    +---> [ Cloud Layer: Firebase Auth + Cloud Firestore (users/{email}/*) ]
    |
    +---> [ Local Layer: Room SQLite Database (user_table, cycle_table) ]
    |
    +---> [ Background: WorkManager (24h Daily, 6h Cycle, 7d Weekly Jobs) ]
```

---

### 3. 20 Critical Definitions (Flashcard Style)

1. **Activity:** A single screen with an interactive user interface (`DashboardActivity`).
2. **Fragment:** A modular, reusable sub-section of an Activity (`TodayFragment`).
3. **Room Database:** An Android Jetpack ORM abstraction layer over native SQLite.
4. **DAO (Data Access Object):** An interface defining SQL operations with compile-time verification (`CycleDao`).
5. **Entity:** A Java model class annotated with `@Entity` representing an SQLite table (`UserEntity`).
6. **WorkManager:** An Android Jetpack library for deferrable, guaranteed background work surviving app kills.
7. **PeriodicWorkRequest:** A recurring background task with a minimum 15-minute interval.
8. **OneTimeWorkRequest:** A background task executed exactly once for immediate operations.
9. **Constraints:** Hardware prerequisites (e.g., `NetworkType.CONNECTED`) required before WorkManager executes.
10. **Partial WakeLock:** A power management lock keeping the CPU running while the screen turns off.
11. **Cloud Firestore:** A scalable, cloud-hosted NoSQL document-collection database.
12. **Firestore Document:** A key-value record residing inside a collection (e.g., `users/hiten@example.com`).
13. **Firestore Collection:** A container of documents (e.g., `users`, `daily_logs`).
14. **FileProvider:** A secure `ContentProvider` generating temporary `content://` URIs for file sharing.
15. **PdfDocument:** Android's native API for drawing multi-page vector PDF documents on a canvas.
16. **Volley:** An asynchronous HTTP networking library managing request queues and REST requests.
17. **SharedPreferences:** An XML-based key-value disk storage for primitive app settings.
18. **ANR (Application Not Responding):** An error triggered when the Main Thread is blocked for $>5$ seconds.
19. **Luteal Phase:** The post-ovulatory phase of the menstrual cycle, biologically constant at $\approx 14$ days.
20. **Fertile Window:** The 6-to-7 day window (5 days pre-ovulation + ovulation day) where pregnancy can occur.

---

### 4. 20 High-Priority Technical Questions Checklist

- [ ] Can you explain the difference between Room and SQLite directly?
- [ ] Can you walk through the roll-forward algorithm in `TodayFragment`?
- [ ] How does `AppDatabase` implement the thread-safe Singleton pattern?
- [ ] What is the difference between `ExistingPeriodicWorkPolicy.KEEP` and `REPLACE`?
- [ ] Why does `ScheduledNotificationWorker` acquire a WakeLock in `MODE_IMMEDIATE`?
- [ ] How do Firestore Security Rules enforce data isolation between users?
- [ ] What happens if an unhandled exception occurs in `FemCareApplication`?
- [ ] Why is `POST_NOTIFICATIONS` runtime permission mandatory on Android 13+?
- [ ] How does `SettingsFragment` generate and share a medical PDF?
- [ ] Why is WorkManager preferred over AlarmManager for health reminders?
- [ ] What are the two tables in the local Room database?
- [ ] What is the difference between `commit()` and `apply()` in SharedPreferences?
- [ ] How does `FindDoctorActivity` combine OpenStreetMap and Google Places?
- [ ] What is the purpose of `wrapCloudSaveWithTimeout` in `TrackFragment`?
- [ ] How does `FirebaseAuthState` prevent retry storms during token failures?
- [ ] What is the status of the Node.js/Express backend in the repository?
- [ ] Why is Retrofit declared in Gradle but not instantiated in Java?
- [ ] What is the purpose of `LocalNotificationStore`?
- [ ] How does `CalendarAdapter` calculate which cells are period days vs. fertile days?
- [ ] What is the difference between `View.GONE` and `View.INVISIBLE`?

---

### 5. 15 Difficult Counter-Questions to Prepare For

1. *"Why didn't you use Kotlin Coroutines instead of Java Executors?"*  
   $\rightarrow$ Prioritized mastering Java fundamentals, threading primitives, and memory architecture first.
2. *"Why is `exportSchema = false` in your Room database?"*  
   $\rightarrow$ Disabled during early prototyping; in production, set to true to track versioned schema JSON.
3. *"What happens if a user enters a cycle length of 10 days?"*  
   $\rightarrow$ Validated in `ScheduledNotificationWorker`: clamped to a realistic baseline of 28 if $<20$ or $>90$.
4. *"Why use `Tasks.await()` instead of callbacks in WorkManager?"*  
   $\rightarrow$ `doWork()` executes synchronously on a background thread and requires a synchronous `Result`.
5. *"Why is your Node.js backend using in-memory arrays?"*  
   $\rightarrow$ Engineered as a rapid prototype for modular API experimentation before adding persistent DB models.
6. *"How do you prevent SQL injection in Room?"*  
   $\rightarrow$ Room uses parameterized queries (`:param`), compiling them into SQLite `PreparedStatement`s.
7. *"Can a user's health notes be read if their phone is stolen?"*  
   $\rightarrow$ Currently stored in app sandbox; in production, SQLCipher would be added to encrypt the database.
8. *"What if the user denies notification permission?"*  
   $\rightarrow$ WorkManager tasks still run; `NotificationHelper` silently suppresses `notify()` to avoid crashes.
9. *"Why not use `AlarmManager.setExactAndAllowWhileIdle`?"*  
   $\rightarrow$ Drains excessive battery; menstrual cycle reminders do not require exact-to-the-second precision.
10. *"What is the risk of using user email as a document ID?"*  
    $\rightarrow$ Emails are mutable and contain special characters; production standard is immutable `request.auth.uid`.
11. *"How does `HexagonProgressBar` draw a hexagon?"*  
    $\rightarrow$ Uses canvas trigonometry calculating 6 points at $60^\circ$ radial intervals ($R\cos\theta, R\sin\theta$).
12. *"What happens if the device reboots?"*  
    $\rightarrow$ WorkManager listens for `ACTION_BOOT_COMPLETED` and automatically restores scheduled tasks.
13. *"How does `CalendarFragment` handle swipe gestures?"*  
    $\rightarrow$ Registers a `GestureDetector.SimpleOnGestureListener` inspecting velocity and X-axis displacement.
14. *"Why do you have two reminder workers (`ScheduledNotificationWorker` and `PeriodReminderWorker`)?"*  
    $\rightarrow$ `PeriodReminderWorker` was an early prototype querying Room; `BackgroundTaskScheduler` now cancels it in favor of the unified `ScheduledNotificationWorker`.
15. *"How do you handle memory leaks in Fragments?"*  
    $\rightarrow$ Clean up view bindings and remove Firestore snapshot listeners in `onDestroyView()`.

---

### 6. 10 Critical Technology Comparisons

1. **Room vs. SQLite:** Room offers compile-time query verification and ORM mapping; raw SQLite risks runtime crashes.
2. **Room vs. SharedPreferences:** Room is a relational SQL database for structured tables; SharedPreferences is flat XML key-value storage.
3. **Firestore vs. Realtime DB:** Firestore supports rich compound querying and subcollections; Realtime DB is a large JSON tree.
4. **WorkManager vs. AlarmManager:** WorkManager is deferrable, battery-conscious, and constraint-based; AlarmManager is exact and battery-heavy.
5. **Foreground Service vs. Worker:** Foreground Services require a persistent notification for user-visible work; Workers run deferrable tasks in the background.
6. **Explicit vs. Implicit Intent:** Explicit names the destination class; Implicit declares a generic OS action (`ACTION_VIEW`).
7. **LiveData vs. StateFlow:** LiveData is lifecycle-aware and tied to Android UI; StateFlow is a pure Kotlin coroutine flow.
8. **Serializable vs. Parcelable:** Serializable uses slow reflection; Parcelable is an Android-optimized high-performance binary protocol.
9. **Authentication vs. Authorization:** Authentication verifies user identity; Authorization controls resource access permissions.
10. **SQL vs. NoSQL:** SQL uses strict relational schemas with ACID guarantees; NoSQL offers flexible document hierarchies with horizontal scaling.

---

### 7. 10 Critical Repository Claims to Verify

1. **Pure Java:** Exactly 39 Java source files in `com.miniflo.femcare`; zero Kotlin files.
2. **compileSdk / targetSdk:** Both set to 36; `minSdk` set to 24.
3. **Room Database Name:** `femcare_database` defined in `AppDatabase.java`.
4. **Room Tables:** `user_table` (`UserEntity`) and `cycle_table` (`CycleEntity`).
5. **Firestore Path:** Root collection `users`, document key `{email}`, subcollections `daily_logs`, `notes`, `notifications`.
6. **WorkManager Periodic Work Names:** `work_daily_log_reminder`, `work_cycle_alerts`, `work_weekly_wellness`.
7. **Immediate Work Name:** `work_immediate_notification_sync`.
8. **Node.js Port:** Port 5000 (`backend/server.js`), running Express with CORS.
9. **Decoupled Backend Reality:** Android app does NOT call Node backend; backend uses in-memory arrays (`let logs = []`).
10. **Retrofit Reality:** Dependency exists in Gradle, but no Java Retrofit interfaces are instantiated.

---

### 8. 5 Professional Spoken Responses When You Don't Know an Answer

1. *"I have not implemented that specific optimization in FemCare, but conceptually I understand that it involves..."*
2. *"In our implementation, we handled that at the application layer using X; however, I recognize that at scale, the industry standard is Y."*
3. *"I am familiar with the concept, but I haven't worked with that specific API directly. I would approach it by reviewing the official Android Jetpack documentation."*
4. *"That is an interesting edge case. In our testing, we didn't encounter that scenario, but to resolve it, I would inspect..."*
5. *"I want to be completely transparent: I haven't implemented that specific pattern myself, but I understand the core principles and am eager to apply it."*

---

# PART 22 — SELF-ASSESSMENT CHECKLIST

Evaluate your interview readiness across all 18 project competencies. Target: **100% Interview-Ready by 2 November 2026.**

| Competency Area | Beginner (Familiar with Concept) | Intermediate (Can Explain Implementation) | Interview-Ready (Can Defend Architecture & Edge Cases) | Candidate Self-Rating |
| :--- | :---: | :---: | :---: | :---: |
| **1. Project Objectives & Problem Statement** | Understands app purpose | Explains 4 core user problems | Defends medical limitations & clinical value | [ ] |
| **2. Android Application Architecture** | Knows Activities & Fragments | Explains Activity-Fragment flow | Defends MVVM vs. Controller choices | [ ] |
| **3. Android SDK & Core Components** | Knows Intent, View, Context | Explains Lifecycle & Permissions | Defends Foreground vs. Background execution | [ ] |
| **4. Java Fundamentals in Android** | Knows OOP definitions | Relates OOP to FemCare classes | Defends Threading, Memory & Concurrency | [ ] |
| **5. Firebase Authentication** | Knows login vs register | Explains Google OAuth & Tokens | Explains token refresh degradation & backoff | [ ] |
| **6. Cloud Firestore NoSQL** | Knows collections & docs | Explains document hierarchy | Defends Firestore Security Rules & schema | [ ] |
| **7. Room Database & SQLite** | Knows SQL vs Room | Explains Entity, DAO, AppDatabase | Defends Double-Checked Locking & Migrations | [ ] |
| **8. Offline-First Architecture** | Knows offline concept | Explains local cache & sync | Explains conflict resolution & eventual consistency | [ ] |
| **9. Cycle Prediction Mathematics** | Knows 28-day baseline | Explains ovulation & fertile math | Explains roll-forward algorithm & biology limits | [ ] |
| **10. Symptom Logging & Analytics** | Knows chip selections | Explains date-keyed documents | Explains MPAndroidChart trends & data parsing | [ ] |
| **11. Interactive Calendar Grid** | Knows 7-column layout | Explains CalendarAdapter math | Explains gesture detection & daily note sheets | [ ] |
| **12. WorkManager Background System** | Knows WorkManager purpose | Explains 3 periodic jobs & modes | Defends WakeLocks, constraints & Doze mode | [ ] |
| **13. System Notifications & Channels** | Knows NotificationManager | Explains NotificationChannel & API 33 | Explains LocalNotificationStore JSON cache | [ ] |
| **14. Node.js & Express Backend** | Knows Node & Express | Explains endpoints & controllers | Honestly explains in-memory arrays & decoupling | [ ] |
| **15. Native PDF Generation** | Knows PDF export | Explains PdfDocument & Canvas | Explains FileProvider secure URI sharing | [ ] |
| **16. Location & Maps Integration** | Knows Google Maps | Explains OSM/Overpass hybrid model | Explains Volley queries & FusedLocationClient | [ ] |
| **17. Privacy, Security & Resilience** | Knows data is sensitive | Explains crash handler & restart | Explains HIPAA/GDPR risks & SQLCipher roadmap | [ ] |
| **18. Honest Technical Defense** | Avoids tough questions | Answers honestly about code | Defends AI-assisted development & ownership | [ ] |

---
*End of FemCare Technical Interview Handbook — Prepared for Hiten Bhurani | 2 November 2026*
