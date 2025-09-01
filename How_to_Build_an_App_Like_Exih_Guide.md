# How to Build a macOS App Like Exih.app: Complete Guide

## Overview
Exih.app is a macOS application wrapper that provides a clean, user-friendly interface to launch Python-based automation scripts. This guide will teach you how to create similar apps that wrap command-line tools or scripts into native macOS applications.

## What You'll Learn
- How to create macOS application bundles (.app)
- How to write shell script executables for app launchers
- How to add custom icons and metadata
- How to handle error cases and user feedback
- Best practices for app distribution

## Prerequisites
- macOS development machine
- Basic knowledge of shell scripting (bash)
- Text editor or IDE
- Understanding of macOS application structure

## Part 1: Understanding macOS App Bundle Structure

A macOS app is actually a special folder called a "bundle" with a `.app` extension. Here's the structure:

```
YourApp.app/
├── Contents/
│   ├── Info.plist          # App metadata and configuration
│   ├── MacOS/
│   │   └── YourApp         # Executable file (can be binary or script)
│   └── Resources/
│       ├── AppIcon.icns    # App icon
│       └── [other resources]
└── Icon\r                  # Optional: custom folder icon
```

## Part 2: Step-by-Step Creation Process

### Step 1: Create the App Bundle Structure

```bash
# Create your app directory
mkdir "MyApp.app"
mkdir "MyApp.app/Contents"
mkdir "MyApp.app/Contents/MacOS"
mkdir "MyApp.app/Contents/Resources"
```

### Step 2: Create the Info.plist File

The `Info.plist` file contains essential metadata about your app:

```xml path=null start=null
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleExecutable</key>
    <string>MyApp</string>
    <key>CFBundleIdentifier</key>
    <string>com.yourname.myapp</string>
    <key>CFBundleName</key>
    <string>MyApp</string>
    <key>CFBundleDisplayName</key>
    <string>My Application</string>
    <key>CFBundleVersion</key>
    <string>1.0</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleSignature</key>
    <string>????</string>
    <key>CFBundleIconFile</key>
    <string>AppIcon</string>
    <key>LSMinimumSystemVersion</key>
    <string>10.9</string>
    <key>LSApplicationCategoryType</key>
    <string>public.app-category.utilities</string>
    <key>NSHighResolutionCapable</key>
    <true/>
</dict>
</plist>
```

**Key Fields Explained:**
- `CFBundleExecutable`: Name of your executable file
- `CFBundleIdentifier`: Unique identifier (reverse domain notation)
- `CFBundleName`: Internal app name
- `CFBundleDisplayName`: Name shown to users
- `LSApplicationCategoryType`: App Store category

### Step 3: Create the Executable Script

Based on Exih.app's approach, create your main executable script:

```bash path=null start=null
#!/bin/bash

# MyApp - Launcher for your scripts/tools
# Description: What your app does

SCRIPT_DIR="/path/to/your/script/directory"
SCRIPT_NAME="your_script.py"  # or .command, .sh, etc.

# Check if the script directory exists
if [ ! -d "$SCRIPT_DIR" ]; then
    osascript -e 'display dialog "Error: Script directory not found at '"$SCRIPT_DIR"'" buttons {"OK"} default button "OK" with icon stop'
    exit 1
fi

# Check if the specific script exists
if [ ! -f "$SCRIPT_DIR/$SCRIPT_NAME" ]; then
    osascript -e 'display dialog "Error: Script not found: '"$SCRIPT_NAME"'" buttons {"OK"} default button "OK" with icon stop'
    exit 1
fi

# Change to script directory
cd "$SCRIPT_DIR"

# Option 1: Run in Terminal (like Exih does)
osascript -e 'tell application "Terminal" to do script "cd \"'"$SCRIPT_DIR"'\" && ./'"$SCRIPT_NAME"'"'

# Option 2: Run silently in background
# python3 "$SCRIPT_NAME" &

# Option 3: Run with output capture
# OUTPUT=$(python3 "$SCRIPT_NAME" 2>&1)
# osascript -e 'display dialog "Script completed. Output: '"$OUTPUT"'" buttons {"OK"} default button "OK"'
```

Save this as `MyApp.app/Contents/MacOS/MyApp` and make it executable:

```bash
chmod +x "MyApp.app/Contents/MacOS/MyApp"
```

### Step 4: Create an App Icon

You need an `.icns` icon file:

**Method 1: Using Preview**
1. Create or find a 1024x1024 PNG image
2. Open in Preview
3. File > Export > Format: Apple Icon Image (.icns)
4. Save as `AppIcon.icns` in `MyApp.app/Contents/Resources/`

**Method 2: Using Command Line**
```bash
# If you have ImageMagick installed
brew install imagemagick
convert your_icon.png -resize 1024x1024 "MyApp.app/Contents/Resources/AppIcon.icns"
```

**Method 3: Using iconutil (Built into macOS)**
```bash
# Create iconset folder with multiple sizes
mkdir MyIcon.iconset
sips -z 16 16     your_icon.png --out MyIcon.iconset/icon_16x16.png
sips -z 32 32     your_icon.png --out MyIcon.iconset/icon_16x16@2x.png
sips -z 32 32     your_icon.png --out MyIcon.iconset/icon_32x32.png
sips -z 64 64     your_icon.png --out MyIcon.iconset/icon_32x32@2x.png
sips -z 128 128   your_icon.png --out MyIcon.iconset/icon_128x128.png
sips -z 256 256   your_icon.png --out MyIcon.iconset/icon_128x128@2x.png
sips -z 256 256   your_icon.png --out MyIcon.iconset/icon_256x256.png
sips -z 512 512   your_icon.png --out MyIcon.iconset/icon_256x256@2x.png
sips -z 512 512   your_icon.png --out MyIcon.iconset/icon_512x512.png
sips -z 1024 1024 your_icon.png --out MyIcon.iconset/icon_512x512@2x.png

# Convert to .icns
iconutil -c icns MyIcon.iconset
mv MyIcon.icns "MyApp.app/Contents/Resources/AppIcon.icns"
```

## Part 3: Advanced Features

### Error Handling with AppleScript Dialogs

Use AppleScript for user-friendly error messages:

```bash path=null start=null
# Success notification
osascript -e 'display notification "Task completed successfully!" with title "MyApp"'

# Warning dialog
osascript -e 'display dialog "Warning: Something needs attention" buttons {"Cancel", "Continue"} default button "Continue" with icon caution'

# Error dialog with custom icon
osascript -e 'display dialog "Error occurred!" buttons {"OK"} default button "OK" with icon stop'

# Input dialog
USER_INPUT=$(osascript -e 'display dialog "Enter configuration:" default answer "" buttons {"Cancel", "OK"} default button "OK"' -e 'text returned of result')
```

### Configuration and Settings

Create a settings system:

```bash path=null start=null
#!/bin/bash

# Configuration file path
CONFIG_DIR="$HOME/.myapp"
CONFIG_FILE="$CONFIG_DIR/settings.conf"

# Create config directory if it doesn't exist
mkdir -p "$CONFIG_DIR"

# Load settings or create defaults
if [ -f "$CONFIG_FILE" ]; then
    source "$CONFIG_FILE"
else
    # Default settings
    SCRIPT_PATH="/path/to/default/script"
    AUTO_CLOSE_TERMINAL=true
    
    # Save defaults
    cat > "$CONFIG_FILE" << EOF
SCRIPT_PATH="$SCRIPT_PATH"
AUTO_CLOSE_TERMINAL=$AUTO_CLOSE_TERMINAL
EOF
fi
```

### Logging

Add logging capabilities:

```bash path=null start=null
LOG_DIR="$HOME/.myapp/logs"
LOG_FILE="$LOG_DIR/myapp.log"

mkdir -p "$LOG_DIR"

log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

log_message "App started"
```

## Part 4: Different Launcher Patterns

### Pattern 1: Terminal-Based (Like Exih)
```bash path=null start=null
# Opens Terminal and runs command
osascript -e 'tell application "Terminal" to do script "cd \"'"$SCRIPT_DIR"'\" && ./'"$SCRIPT_NAME"'"'
```

### Pattern 2: Silent Background Execution
```bash path=null start=null
# Runs silently without showing Terminal
cd "$SCRIPT_DIR"
python3 "$SCRIPT_NAME" > /dev/null 2>&1 &
```

### Pattern 3: Capture Output and Display
```bash path=null start=null
# Capture output and show in dialog
cd "$SCRIPT_DIR"
OUTPUT=$(python3 "$SCRIPT_NAME" 2>&1)
osascript -e 'display dialog "Result: '"$OUTPUT"'" buttons {"OK"} default button "OK"'
```

### Pattern 4: Web App Launcher
```bash path=null start=null
# Launch a local web server and open browser
cd "$SCRIPT_DIR"
python3 -m http.server 8000 &
sleep 2
open "http://localhost:8000"
```

## Part 5: Testing Your App

1. **Test the executable directly:**
```bash
"/path/to/YourApp.app/Contents/MacOS/YourApp"
```

2. **Test double-click functionality:**
   - Double-click the .app bundle in Finder
   - Verify it behaves as expected

3. **Test error conditions:**
   - Remove the target script temporarily
   - Verify error dialogs appear correctly

## Part 6: Distribution and Installation

### For Personal Use
Simply copy the `.app` bundle to your Applications folder:
```bash
cp -R "MyApp.app" "/Applications/"
```

### For Distribution
1. **Code Signing (Recommended):**
```bash
# Sign the app (requires Apple Developer account)
codesign --force --deep --sign "Developer ID Application: Your Name" "MyApp.app"
```

2. **Create a DMG:**
```bash
# Create a disk image for distribution
hdiutil create -volname "MyApp" -srcfolder "MyApp.app" -ov -format UDZO "MyApp.dmg"
```

3. **Notarization (For Gatekeeper):**
```bash
# Submit for notarization (requires Apple Developer account)
xcrun notarytool submit "MyApp.dmg" --keychain-profile "notarytool-profile" --wait
```

## Part 7: Example Projects You Can Build

### 1. Development Tool Launcher
```bash path=null start=null
#!/bin/bash
# Launches your development environment

osascript -e 'tell application "Terminal" to do script "cd ~/projects/myproject && code . && npm start"'
```

### 2. System Maintenance App
```bash path=null start=null
#!/bin/bash
# Runs system cleanup scripts

osascript -e 'display dialog "Run system maintenance?" buttons {"Cancel", "Run"} default button "Run"'
if [ $? -eq 0 ]; then
    # Run maintenance scripts
    sudo periodic daily weekly monthly
    osascript -e 'display notification "Maintenance completed!" with title "System Maintenance"'
fi
```

### 3. Backup Launcher
```bash path=null start=null
#!/bin/bash
# Launches backup scripts

BACKUP_SCRIPT="$HOME/scripts/backup.sh"
osascript -e 'tell application "Terminal" to do script "'"$BACKUP_SCRIPT"'"'
```

## Part 8: Advanced Tips

### 1. Dynamic Script Selection
```bash path=null start=null
# Let user choose which script to run
SCRIPTS_DIR="/path/to/scripts"
SCRIPT_LIST=$(ls "$SCRIPTS_DIR"/*.py | xargs -n 1 basename)
CHOSEN_SCRIPT=$(osascript -e 'choose from list {"'$(echo $SCRIPT_LIST | sed 's/ /", "/g')'} with prompt "Select script to run:"')
```

### 2. Environment Setup
```bash path=null start=null
# Set up environment before running
export PATH="/usr/local/bin:$PATH"
export PYTHONPATH="/path/to/your/modules:$PYTHONPATH"
source "$HOME/.venv/bin/activate"  # Activate virtual environment
```

### 3. Cross-Platform Compatibility
```bash path=null start=null
# Detect the operating system
if [[ "$OSTYPE" == "darwin"* ]]; then
    # macOS specific code
    open -a Terminal
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
    # Linux specific code
    gnome-terminal --
fi
```

## Part 9: Troubleshooting Common Issues

### Issue 1: Permission Denied
**Solution:** Make sure your executable has the right permissions:
```bash
chmod +x "YourApp.app/Contents/MacOS/YourApp"
```

### Issue 2: App Won't Launch
**Checklist:**
- Info.plist CFBundleExecutable matches your script name
- Executable file exists and has correct permissions
- No syntax errors in your shell script

### Issue 3: Gatekeeper Blocks App
**Solutions:**
- Right-click app, select "Open" to bypass once
- System Preferences > Security & Privacy > Allow apps downloaded from: Anywhere
- Code sign your app (preferred for distribution)

### Issue 4: Script Not Found
**Add robust path checking:**
```bash path=null start=null
SCRIPT_PATH="/path/to/script"
if [ ! -f "$SCRIPT_PATH" ]; then
    osascript -e 'display dialog "Script not found. Please check installation." buttons {"OK"} with icon stop'
    exit 1
fi
```

## Part 10: Complete Example - "QuickServer App"

Let's build a complete example that launches a local web server:

### 1. Create the structure:
```bash
mkdir "QuickServer.app"
mkdir "QuickServer.app/Contents"
mkdir "QuickServer.app/Contents/MacOS"
mkdir "QuickServer.app/Contents/Resources"
```

### 2. Info.plist:
```xml path=null start=null
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleExecutable</key>
    <string>QuickServer</string>
    <key>CFBundleIdentifier</key>
    <string>com.yourname.quickserver</string>
    <key>CFBundleName</key>
    <string>QuickServer</string>
    <key>CFBundleDisplayName</key>
    <string>Quick Server</string>
    <key>CFBundleVersion</key>
    <string>1.0</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundlePackageType</key>
    <string>APPL</string>
    <key>CFBundleIconFile</key>
    <string>AppIcon</string>
    <key>LSApplicationCategoryType</key>
    <string>public.app-category.developer-tools</string>
    <key>NSHighResolutionCapable</key>
    <true/>
</dict>
</plist>
```

### 3. Executable script:
```bash path=null start=null
#!/bin/bash

# QuickServer - Launch a local development server

SERVER_DIR=$(osascript -e 'POSIX path of (choose folder with prompt "Select folder to serve:")')

if [ -z "$SERVER_DIR" ]; then
    exit 0  # User cancelled
fi

# Remove trailing newline
SERVER_DIR=$(echo "$SERVER_DIR" | tr -d '\n')

# Get port number
PORT=$(osascript -e 'display dialog "Enter port number:" default answer "8000" buttons {"Cancel", "Start Server"} default button "Start Server"' -e 'text returned of result')

if [ -z "$PORT" ]; then
    exit 0  # User cancelled
fi

# Start server and open browser
osascript -e 'tell application "Terminal" to do script "cd \"'"$SERVER_DIR"'\" && python3 -m http.server '"$PORT"' && echo \"Server stopped. Close this tab.\""'

sleep 3  # Give server time to start
open "http://localhost:$PORT"

# Show notification
osascript -e 'display notification "Server started on port '"$PORT"'" with title "QuickServer"'
```

### 4. Make executable:
```bash
chmod +x "QuickServer.app/Contents/MacOS/QuickServer"
```

## Part 11: Best Practices

### 1. User Experience
- Always provide clear error messages
- Use native macOS dialogs for consistency
- Include progress indicators for long operations
- Provide helpful notifications

### 2. Security
- Validate all user inputs
- Use absolute paths to avoid security issues
- Don't hardcode sensitive information
- Consider code signing for distribution

### 3. Maintenance
- Include version information in your app
- Add logging for debugging
- Create clear documentation
- Use configuration files for flexibility

### 4. Professional Polish
- Create proper app icons at multiple resolutions
- Write comprehensive error handling
- Add support for command-line arguments
- Include help documentation

## Part 12: Distribution Checklist

Before distributing your app:

- [ ] Test on clean macOS system
- [ ] Verify all dependencies are available
- [ ] Code sign the application
- [ ] Create installer package if needed
- [ ] Write user documentation
- [ ] Test with different macOS versions
- [ ] Prepare support resources

## Conclusion

You now have all the knowledge needed to create macOS app wrappers like Exih.app! The key is understanding that macOS apps are just specially structured folders with metadata, an executable, and resources.

Start with a simple app like the QuickServer example, then gradually add more features as you become comfortable with the process.

## Additional Resources

- [Apple's Bundle Programming Guide](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/)
- [Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/)
- [AppleScript Language Guide](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/)

Happy app building!
