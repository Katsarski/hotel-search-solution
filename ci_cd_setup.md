This document explains on high level how test automation can be integrated into CI/CD

Option 1
stages:
  - checkout hotel-search app
  - install dependencies (emulators, flutter etc.)
  - build (for Android, iOS etc.)
  - setup and spin up emulator
  - run the build artifact on emulator
  - run tests against app
  
Option 2
stages:
  - pull the artifact to be tested (APK/IPA file)
  - install dependencies (emulators, maestro etc.)
  - setup and spin up emulator
  - launch the artifact onto the emulator
  - run tests against app