name: Build Android APK

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y build-essential git python3-dev \
            libssl-dev libffi-dev libgl1-mesa-dev libgles2-mesa-dev \
            zlib1g-dev openjdk-11-jdk wget unzip
          pip install --upgrade pip
          pip install kivy buildozer

      - name: Install Android SDK & NDK
        run: |
          wget https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip -O cmdline-tools.zip
          unzip cmdline-tools.zip -d $HOME/android-sdk
          yes | $HOME/android-sdk/cmdline-tools/bin/sdkmanager --sdk_root=$HOME/android-sdk "platform-tools" "platforms;android-29" "build-tools;29.0.3" "ndk;21.4.7075529"

      - name: Configure environment
        run: |
          echo "export ANDROID_HOME=$HOME/android-sdk" >> $GITHUB_ENV
          echo "export ANDROID_NDK_HOME=$HOME/android-sdk/ndk/21.4.7075529" >> $GITHUB_ENV
          echo "export PATH=$PATH:$HOME/android-sdk/platform-tools:$HOME/android-sdk/cmdline-tools/bin" >> $GITHUB_ENV

      - name: Build APK with Buildozer
        run: |
          buildozer android debug
      
      - name: Upload APK
        uses: actions/upload-artifact@v3
        with:
          name: whatsappbulk-apk
          path: bin/WhatsAppBulkSender-0.1-debug.apk
