name: Build APK

on:
  workflow_dispatch:   # se ejecuta manualmente
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install --legacy-peer-deps

      - name: Build web
        run: npm run build

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Setup Android SDK
        uses: android-actions/setup-android@v3

      - name: Add Capacitor Android
        run: |
          npm install @capacitor/core @capacitor/cli @capacitor/android
          npx cap add android
          npx cap sync

      - name: Build Debug APK
        working-directory: android
        run: ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: NominaFacil-APK
          path: android/app/build/outputs/apk/debug/app-debug.apk