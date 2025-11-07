# EclipseOS OTA Upload Guide for Maintainers

This guide explains how to upload new OTA updates for your device to the EclipseOS OTA repository.

## 📋 Prerequisites

- Maintainer access to https://github.com/ProjectEclipseOfi/ota
- Built EclipseOS ROM for your device
- Git installed on your system
- GitHub account with write access to the OTA repo

## 🚀 Quick Start

### 1. Clone the OTA Repository

```bash
# Clone the repository
git clone https://github.com/ProjectEclipseOfi/ota.git
cd ota

# Switch to eclipse-16 branch
git checkout eclipse-16

# Pull latest changes
git pull origin eclipse-16
```

### 2. Prepare Your ROM Build

After building your ROM, gather the following information:

```bash
# Set your device codename
DEVICE="beryllium"  # Change this to your device

# Your ROM file
ROM_FILE="EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip"

# Get MD5 hash
MD5=$(md5sum $ROM_FILE | awk '{print $1}')
echo "MD5: $MD5"

# Get file size in bytes
SIZE=$(stat -c%s "$ROM_FILE" 2>/dev/null || stat -f%z "$ROM_FILE")
echo "Size: $SIZE bytes"

# Get current timestamp
TIMESTAMP=$(date +%s)
echo "Timestamp: $TIMESTAMP"

# Get human-readable date
DATE=$(date +%Y%m%d)
echo "Date: $DATE"
```

### 3. Create/Update Device Folder

```bash
# If this is your first upload, create device folder
mkdir -p $DEVICE

# Navigate to your device folder
cd $DEVICE
```

### 4. Update the update.json File

Create or edit `update.json`:

```bash
nano update.json
```

**Template:**
```json
{
  "response": [
    {
      "datetime": 1730995200,
      "filename": "EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip",
      "id": "abc123def456789",
      "romtype": "OFFICIAL",
      "size": 1073741824,
      "url": "https://github.com/ProjectEclipseOfi/ota/releases/download/v1.0/EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip",
      "version": "1.0"
    }
  ]
}
```

**Field Explanations:**

| Field | Description | Example |
|-------|-------------|---------|
| `datetime` | Build date as UNIX timestamp | `1730995200` |
| `filename` | Exact name of your ROM ZIP | `EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip` |
| `id` | MD5 hash of the ROM file | `abc123def456789` |
| `romtype` | Build type (OFFICIAL/UNOFFICIAL/NIGHTLY) | `OFFICIAL` |
| `size` | File size in bytes | `1073741824` |
| `url` | Direct download URL from GitHub Releases | See below |
| `version` | ROM version number | `1.0` |

### 5. Update the Changelog

Create or edit `changelog.md`:

```bash
nano changelog.md
```

**Template:**
```markdown
# EclipseOS Changelog - [Device Name]

## Version 1.0 - November 7, 2024

### New Features
- Initial EclipseOS release
- Privacy-first features enabled by default
- Custom EclipseOS Launcher
- Enhanced Settings app with device info
- Automatic OTA updates

### Improvements
- Based on Android 16 (LineageOS 23.0)
- Optimized performance and battery life
- Improved system stability
- Better memory management

### Bug Fixes
- Fixed camera issues
- Resolved Bluetooth connectivity
- Fixed notification problems

### Known Issues
- List any known issues here
- Will be fixed in next update

### Device Specific
- Add device-specific changes here
- Kernel version, patches, etc.

---

**Maintainer**: Your Name  
**Build Date**: November 7, 2024  
**Build Type**: OFFICIAL
```

### 6. Create GitHub Release

1. **Go to Releases**: https://github.com/ProjectEclipseOfi/ota/releases

2. **Click "Draft a new release"**

3. **Fill in release details**:
   - **Tag**: `v1.0-beryllium-20241107` (format: `v{version}-{device}-{date}`)
   - **Target**: `eclipse-16` branch
   - **Title**: `EclipseOS 1.0 - Beryllium (Poco F1) - November 7, 2024`
   - **Description**:
     ```markdown
     # EclipseOS 1.0 - Beryllium (Poco F1)
     
     **Device**: Xiaomi Poco F1 (beryllium)  
     **Android Version**: 16  
     **Build Date**: November 7, 2024  
     **Build Type**: OFFICIAL  
     **Maintainer**: Your Name
     
     ## Installation
     - Download the ROM ZIP file below
     - Install via EclipseOS Updater app or recovery
     - Wipe data recommended for first install
     
     ## Changelog
     See changelog.md in the device folder
     
     ## MD5 Checksum
     `abc123def456789`
     
     ## Download
     File size: ~1GB
     ```

4. **Upload ROM file**: Drag and drop your `EclipseOS-*.zip` file

5. **Click "Publish release"**

6. **Copy the download URL**: Right-click on the uploaded file → Copy link address

   Example: `https://github.com/ProjectEclipseOfi/ota/releases/download/v1.0-beryllium-20241107/EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip`

### 7. Update JSON with Release URL

Go back to your `update.json` and update the `url` field with the copied link:

```json
{
  "response": [
    {
      "datetime": 1730995200,
      "filename": "EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip",
      "id": "abc123def456789",
      "romtype": "OFFICIAL",
      "size": 1073741824,
      "url": "https://github.com/ProjectEclipseOfi/ota/releases/download/v1.0-beryllium-20241107/EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip",
      "version": "1.0"
    }
  ]
}
```

### 8. Commit and Push Changes

```bash
# Go back to ota repo root
cd /path/to/ota

# Add your changes
git add $DEVICE/update.json $DEVICE/changelog.md

# Commit with descriptive message
git commit -m "Update $DEVICE to v1.0 ($(date +%Y-%m-%d))"

# Push to GitHub
git push origin eclipse-16
```

### 9. Verify the Update

Test that your update is accessible:

```bash
# Check if JSON is accessible
curl https://raw.githubusercontent.com/ProjectEclipseOfi/ota/eclipse-16/$DEVICE/update.json

# Should return your JSON content
```

## 📱 Testing on Device

### Via ADB:
```bash
# Open Updater app
adb shell am start -n org.lineageos.updater/.UpdatesActivity

# Check logs
adb logcat | grep Updater

# Force refresh
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED
```

### On Device:
1. Open **Updater** app
2. Tap **Refresh** button
3. Your update should appear
4. Test download and installation

## 🔄 Updating to a New Version

When releasing a new version:

```bash
# Pull latest changes
cd ota
git pull origin eclipse-16

# Update your device files
cd $DEVICE

# Edit update.json with new version info
nano update.json

# Update changelog
nano changelog.md

# Create new GitHub release (follow step 6)

# Commit and push
git add .
git commit -m "Update $DEVICE to v1.1 ($(date +%Y-%m-%d))"
git push origin eclipse-16
```

## 📝 Naming Conventions

### ROM File Naming:
```
EclipseOS-{version}-{device}-{buildtype}-{date}.zip
```
Example: `EclipseOS-1.0-beryllium-OFFICIAL-20241107.zip`

### Release Tag Naming:
```
v{version}-{device}-{date}
```
Example: `v1.0-beryllium-20241107`

### Build Types:
- `OFFICIAL` - Official maintainer builds
- `UNOFFICIAL` - Community/test builds
- `NIGHTLY` - Automated nightly builds
- `BETA` - Beta testing builds

## 🛠️ Automation Script

Create a helper script `upload-ota.sh`:

```bash
#!/bin/bash

# EclipseOS OTA Upload Helper Script

DEVICE="$1"
ROM_FILE="$2"
VERSION="$3"

if [ -z "$DEVICE" ] || [ -z "$ROM_FILE" ] || [ -z "$VERSION" ]; then
    echo "Usage: ./upload-ota.sh <device> <rom_file> <version>"
    echo "Example: ./upload-ota.sh beryllium EclipseOS-1.0-beryllium.zip 1.0"
    exit 1
fi

# Get file info
MD5=$(md5sum "$ROM_FILE" | awk '{print $1}')
SIZE=$(stat -c%s "$ROM_FILE")
TIMESTAMP=$(date +%s)
DATE=$(date +%Y%m%d)
FILENAME=$(basename "$ROM_FILE")

echo "Device: $DEVICE"
echo "File: $FILENAME"
echo "MD5: $MD5"
echo "Size: $SIZE bytes"
echo "Timestamp: $TIMESTAMP"
echo ""
echo "Next steps:"
echo "1. Create GitHub release: v$VERSION-$DEVICE-$DATE"
echo "2. Upload file: $ROM_FILE"
echo "3. Copy download URL"
echo "4. Update update.json with the URL"
echo "5. Run: git add $DEVICE && git commit -m 'Update $DEVICE to v$VERSION' && git push"
```

Make it executable:
```bash
chmod +x upload-ota.sh
```

Use it:
```bash
./upload-ota.sh beryllium EclipseOS-1.0-beryllium-OFFICIAL.zip 1.0
```

## ⚠️ Important Notes

1. **Always test** your ROM before uploading
2. **Verify MD5** matches the uploaded file
3. **Check file size** is correct in bytes
4. **Test the update** on a real device before announcing
5. **Keep changelog updated** with all changes
6. **Use HTTPS URLs** only
7. **Don't delete old releases** unless necessary
8. **Announce updates** in Discord/Telegram after upload

## 🆘 Troubleshooting

### Update not showing in app:
- Check JSON syntax: https://jsonlint.com/
- Verify URL is accessible
- Check device codename matches `ro.eclipse.device`
- Wait 5-10 minutes for GitHub CDN to update

### Download fails:
- Verify GitHub release is published (not draft)
- Check file URL is correct
- Ensure file size matches

### Installation fails:
- Verify ROM is signed correctly
- Check build.prop properties match
- Ensure recovery is compatible

## 📞 Support

If you need help:
- **Discord**: https://discord.gg/Gvduk5QpZp
- **GitHub Issues**: https://github.com/ProjectEclipseOfi/ota/issues

---

**EclipseOS** - Privacy First • Performance Optimized • Developer Friendly
