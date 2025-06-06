# 1. QA environment setup
The following document outlines the procedure for setting up the QA environment on a Windows machine and running some example automated tests, while most steps might be the same for other OSs this has to be further confirmed  

**Prerequisites/Requirements:**  
Tested on/using:  
- Edition	Windows 10 Pro, Version	2009, OS Build	19045.5854  
- Flutter SDK Flutter - 3.32.2/Tools • Dart 3.8.1 • DevTools 2.45.1 (latest stable version)  
- Android Studio - Meerkat Feature Drop | 2024.3.2 Patch 1 (for emulator/device)  
- VS Code - 1.100.3 or Android Studio (for editing and debugging)  
- Java - 1.8.0_451  
- Maestro - 1.40.3  
- Hotel search app  
- A physical Android device or an Android emulator - e.g. through Android Device Manager  
- Dart & Flutter extensions  
- SerpAPI registration and API key  

## Setup:  
- Follow the instructions mentioned here: https://github.com/Buenro/hotel-search i.e.:  
- Run: `git clone https://github.com/akshdeep-singh/hotel_booking`  
- Run: `cd hotel_booking`  
- Run: `flutter pub get`  
- Generate an API key for the SerpAPI service at: https://serpapi.com/manage-api-key  
- Create an .env file in the root of the project with the following content: `SERPAPI_API_KEY={your_api_key}`  
- Run: `flutter pub run build_runner build --delete-conflicting-outputs` - to generate necessary files using the build_runner package  

**Run Emulator:**  
- From Android Studio: Go to Tools > Device Manager > Start your Android emulator.  

From VS Code terminal:  
- Run: `flutter emulators`  
- Run: `flutter emulators --launch <emulator_id>`  

Verify emulator state:  
- Run: `flutter devices`  
- Run: `flutter run`  

**Issues:**  
When following the above-mentioned setup I got build failure during the last step:  
```
Execution failed for task ':path_provider_android:compileDebugJavaWithJavac'.
> Could not resolve all files for configuration ':path_provider_android:androidJdkImage'.
   > Failed to transform core-for-system-modules.jar to match attributes {artifactType=_internal_android_jdk_image, org.gradle.libraryelements=jar, org.gradle.usage=java-runtime}.
      > Execution failed for JdkImageTransform: C:\Users\bo\AppData\Local\Android\sdk\platforms\android-34\core-for-system-modules.jar.
         > Error while executing process C:\Program Files\Android\Android Studio\jbr\bin\jlink.exe with arguments {--module-path C:\Users\bo\.gradle\caches\transforms-3\7ce45b6a92e9356394f32046e3d98b65\transformed\output\temp\jmod --add-modules java.base --output C:\Users\bo\.gradle\caches\transforms-3\7ce45b6a92e9356394f32046e3d98b65\transformed\output\jdkImage --disable-plugin system-modules}

Flutter Fix ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ [!] This is likely due to a known bug in Android Gradle Plugin (AGP) versions less than 8.2.1, when                                   │
│   1. setting a value for SourceCompatibility and                                                                                      │
│   2. using Java 21 or above.                                                                                                          │
│ To fix this error, please upgrade your AGP version to at least 8.2.1. The version of AGP that your project uses is likely defined in: │
│ in the 'plugins' closure (by the number following "com.android.application").                                                         │
│  Alternatively, if your project was created with an older version of the templates, it is likely                                      │
│ in the buildscript.dependencies closure of the top-level build.gradle:                                                                │
│ C:\repos\hotel-search\android\build.gradle,                                                                                           │
│ as the number following "com.android.tools.build:gradle:".                                                                            │
│                                                                                                                                       │
│ For more information, see:                                                                                                            │
│ https://issuetracker.google.com/issues/294137077                                                                                      │
│ https://github.com/flutter/flutter/issues/156304  
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**Workaround:** Following the proposed fix (update the com.android.application version from the initial 8.1.0 to 8.2.1) did cause the application to successfully run, either the setup procedure needs to be updated in the repo readme.md or the version of the Android Gradle Plugin needs to be bumped up to a higher one in the settings.gradle file  

**Potential issue:**  
It might happen that after the setup the emulator does not have access to the internet (making the hotel app unusable)  

**Workaround:** Following the steps mentioned here worked for me: https://stackoverflow.com/a/42784657  


**Example command to run automated tests**  
- Run (from the dir named test inside this repo): `maestro test --format junit ./`    

<br>
<br>
<br>

# 2. QA Test Strategy & Execution Plan  
**Assumptions:**  
- Hotel details page is not yet implemented  
- No hotels/details appear in the Overview page  
- No details appear in the Account page  

**High-level Positive Scenarios to test:**  
- App Launch/Terminate - Ensure the app loads/closes successfully  
- Hotel Search - Input search terms and verify hotel listings matching the search are displayed (high amounts of test data needed), clear search using X  
- Favorites - Add/Remove a Hotel to favorites and verify this is reflected in the Favorites/Hotels tab (variying test data - hotels with short/long names/descriptions)  

**High-level UI Test cases:**  
- Verify all labels, buttons, and images are correctly rendered  
- Verify fonts, colors, and margins match the designss, and good practices like the WCAG are followed  
- Verify all interactive elements are clickable/tappable and accessible/leading to corresponding view  
- Verify no overlapping elements on the screen  
- Verify potrait and landscape modes correctly display each page  
- Verify the UI responsiveness on different screen sizes and devices  
- Verify there is a consistency of style across screens (e.g. button styles, headers)  
- Verify layout adjusts properly for tablet and devices with foldable screens  
- Verify loading states are indicated (e.g. endless spinners/placeholders)  

**High-level App State Transition cases:**  
- Navigating to/from Overview -> Hotels -> Overview -> Favorites -> etc. and verifying that the states (e.g. previous search performed) are persisted  
- Back navigation from the different views (asserting persistance of state if required by design)  
- Reopening app after user puts it in background (app state persistence is retsored)  

**Edge and negative cases:**  
- Search for hotels using long strings (test for buffer overflow, UI freezes, memory leaks)  
- SQL injection against the input fields (hotel search)  
- Add hunders of entries in the Favorites tab  
- Search using special symbols (!@#%, emojis, JSON, and e.g. cyrilic) and see if this impacts the search results and if such are found if present in results  
- Check the app generated data on the device and if it exposes some sensitive information (API keys, secrets)  
- Missing/Wrong API key  
- No search results  
- Network failures  
- Insufficient disk space  

<br>
<br>
<br>

# 3. CI/CD Setup  
This document explains on high level how test automation can be integrated into CI/CD  

**Option 1**  
stages:  
  - checkout hotel-search app  
  - install dependencies (emulators, flutter etc.)  
  - build (for Android, iOS etc.)  
  - setup and spin up emulator  
  - run the build artifact on emulator  
  - run tests against app  
    
**Option 2**  
stages:  
  - pull the artifact to be tested (APK/IPA file)  
  - install dependencies (emulators, maestro etc.)  
  - setup and spin up emulator  
  - launch the artifact onto the emulator  
  - run tests against app  
