# Quick Start Guide - Recording Features

## 📹 Getting Started with Video Recording

### Step 1: Upload to ESP32
1. Open `CameraWebServer.ino` in Arduino IDE
2. Select your ESP32 board and COM port
3. Click **Upload**
4. Wait for upload to complete

### Step 2: Access the Camera
1. Open Serial Monitor (Tools → Serial Monitor)
2. Look for WiFi connection message with IP address
3. Example: `192.168.1.100`

### Step 3: Start Recording

#### Option A: Web UI (Easiest)
1. Open browser: `http://192.168.1.100:80/recording`
2. Select mode:
   - **Continuous**: Records until you stop it
   - **Timed**: Enter hours (1-720)
3. Click **START RECORDING**
4. Click **STOP RECORDING** when done

#### Option B: API Call (Programmatic)
```bash
# Start 1-hour recording
curl "http://192.168.1.100:80/recordingcontrol?action=start&duration=1"

# Stop recording
curl "http://192.168.1.100:80/recordingcontrol?action=stop"

# Check status
curl "http://192.168.1.100:80/recordingstatus"
```

### Step 4: Retrieve Videos

1. **Connect SD Card to PC**
   - Turn off ESP32
   - Remove SD card
   - Insert into card reader

2. **Find Videos**
   - Navigate to root folder
   - Files named: `video_YYYYMMDD_HHMMSS.mjpeg`
   - Example: `video_20260301_143022.mjpeg`

3. **Play Videos**
   - Open with VLC Media Player
   - Or convert to MP4:
     ```bash
     ffmpeg -i video_20260301_143022.mjpeg -c:v libx264 output.mp4
     ```

### Step 5: Monitor Recording

The status display shows:
- 🟢 **Green (Idle)**: No recording
- 🔴 **Red (Recording)**: Recording in progress

Status updates automatically every 2 seconds.

---

## 🎯 Common Tasks

### Continuous Recording (Don't Stop)
1. Select: **Continuous Recording**
2. Click: **START RECORDING**
3. Records until:
   - You click **STOP RECORDING**
   - Device loses power

### Timed Recording (2 Hours)
1. Select: **Timed Recording**
2. Enter: `2`
3. Click: **START RECORDING**
4. Automatically stops after 2 hours

### Check If Recording
Visit: `http://192.168.1.100:80/recordingstatus`

Response:
```json
{"recording": true}   // Currently recording
{"recording": false}  // Idle
```

### Auto-Start Recording on Boot
Edit `CameraWebServer.ino`, find this line:
```cpp
// Start recording automatically (optional)
//startRecording(1);
```

Uncomment it:
```cpp
// Start recording automatically (optional)
startRecording(1);  // Records 1 hour on startup
```

---

## ⚠️ Important Notes

### SD Card Requirements
- **Format**: FAT32
- **Size**: Minimum 8GB (recommended 32GB)
- **Speed**: Class 10 or better

### Storage Capacity
- **32GB card** = ~45-70 hours of recording
- **16GB card** = ~22-35 hours of recording
- **8GB card** = ~11-17 hours of recording

### Power Requirements
- Ensure stable power supply
- Recording on battery: 6+ hours for 32GB
- Use 5V/2A power adapter recommended

### Video Specs
- **Format**: MJPEG
- **FPS**: ~5 frames per second
- **File Size**: ~500-800 KB/min

---

## 🔧 Troubleshooting

### Problem: Recording Won't Start
**Solution:**
1. Check SD card is inserted and formatted (FAT32)
2. Verify sufficient disk space
3. Restart the device
4. Check Serial Monitor for errors

### Problem: Videos Not Saving
**Solution:**
1. Verify SD card is properly mounted:
   - Check Serial Monitor for: "SD Card Size: XXX MB"
2. Try formatting SD card to FAT32
3. Use a different SD card to test
4. Check for bad sectors on card

### Problem: Slow Recording/Frame Drops
**Normal behavior:**
- Recording is set to ~5 FPS for reliability
- This is intentional for stable SD card writing
- To increase FPS:
  1. Edit `CameraWebServer.ino`
  2. Find: `delay(200);` in `recordVideoTask()`
  3. Change to: `delay(100);` for 10 FPS
  4. Re-upload

### Problem: SD Card Not Detected
**Solution:**
1. Verify pin connections (check `camera_pins.h`)
2. Try different SD card (might be card issue)
3. Ensure 1-wire SPI mode for SD_MMC
4. Check power supply to card

### Problem: Browser Can't Access `/recording`
**Solution:**
1. Verify ESP32 IP: `http://192.168.1.100:80`
2. Try: `http://192.168.1.100`
3. Check WiFi connection
4. Restart device after upload
5. Clear browser cache

---

## 📊 Recording Quality Settings

### Default Quality (Recommended)
- **FPS**: 5
- **Resolution**: SVGA (800x600) or VGA depending on card
- **JPEG Quality**: 12-15
- **File Size**: ~10-15 MB/min

### Higher Quality (Larger Files)
Edit `CameraWebServer.ino` in `setup()`:
```cpp
config.jpeg_quality = 8;  // Lower = higher quality, larger files
```

### Lower Quality (Smaller Files)
```cpp
config.jpeg_quality = 20;  // Higher = lower quality, smaller files
```

---

## 🔐 Security Considerations

### Default Setup
- No authentication on recording endpoints
- SD card data unencrypted
- Local network only (recommended)

### Production Use
1. Add authentication to API endpoints
2. Use HTTPS (requires certificate)
3. Restrict WiFi network access
4. Consider encryption for sensitive videos

---

## 📱 API Examples

### Python
```python
import requests

camera = "192.168.1.100"

# Start recording
requests.get(f"http://{camera}:80/recordingcontrol?action=start&duration=2")

# Stop recording
requests.get(f"http://{camera}:80/recordingcontrol?action=stop")

# Check status
status = requests.get(f"http://{camera}:80/recordingstatus").json()
print(f"Recording: {status['recording']}")
```

### JavaScript
```javascript
const camera = "192.168.1.100";

// Start recording
fetch(`http://${camera}:80/recordingcontrol?action=start&duration=1`)
  .then(r => r.json())
  .then(d => console.log("Recording started"));

// Stop recording
fetch(`http://${camera}:80/recordingcontrol?action=stop`)
  .then(r => r.json())
  .then(d => console.log("Recording stopped"));

// Check status
setInterval(() => {
  fetch(`http://${camera}:80/recordingstatus`)
    .then(r => r.json())
    .then(d => console.log(`Recording: ${d.recording}`));
}, 2000);
```

### cURL
```bash
# List all commands
echo "Start (continuous): curl http://192.168.1.100:80/recordingcontrol?action=start&duration=0"
echo "Start (2 hours):    curl http://192.168.1.100:80/recordingcontrol?action=start&duration=2"
echo "Stop:               curl http://192.168.1.100:80/recordingcontrol?action=stop"
echo "Status:             curl http://192.168.1.100:80/recordingstatus"
```

---

## 📚 Additional Resources

- **Full Documentation**: See `RECORDING_FEATURES.md`
- **Change Log**: See `CHANGELOG.md`
- **Video Player**: Install VLC from https://www.videolan.org
- **FFmpeg**: Install from https://ffmpeg.org/download.html
- **Arduino IDE**: https://www.arduino.cc/en/software

---

## ✅ Checklist Before Using

- [ ] ESP32 camera board working and connected
- [ ] 32GB SD card inserted and formatted FAT32
- [ ] Stable 5V power supply
- [ ] Device on same WiFi network as access device
- [ ] Uploaded latest code to ESP32
- [ ] Can access `http://<your-ip>/` for camera feed
- [ ] Checked Serial Monitor for IP address

---

## 🚀 You're Ready!

Your ESP32 camera now has:
- ✅ Full video recording capability
- ✅ Flexible timing (continuous or timed)
- ✅ Web UI for easy control
- ✅ API endpoints for automation
- ✅ Reliable SD card saving
- ✅ Real-time status monitoring

**Happy recording! 🎥**

---

**Last Updated**: March 2026
**Works With**: ESP32, 32GB SD Card, Arduino IDE 1.8.x+
