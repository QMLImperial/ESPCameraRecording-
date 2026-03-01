# ESP32 Camera Recording Features

## Overview
This project now includes a complete video recording system for your ESP32 camera with a 32GB SD card. Videos are recorded as MJPEG files and automatically saved to the SD card.

## Features Added

### 1. **Recording Control UI**
Access the recording interface at: `http://<your-camera-ip>:80/recording`

The UI provides:
- **Start/Stop Recording buttons** for immediate control
- **Two recording modes:**
  - **Continuous Recording**: Records indefinitely until manually stopped
  - **Timed Recording**: Records for a specified duration (1-720 hours)
- **Real-time status display** showing if recording is active or idle
- **Live status updates** every 2 seconds

### 2. **HTTP API Endpoints**

#### Get Recording Status
```
GET /recordingstatus
```
Returns JSON response:
```json
{"recording": true}  // or false
```

#### Control Recording
```
GET /recordingcontrol?action=start&duration=<hours>
GET /recordingcontrol?action=stop
```

**Parameters:**
- `action`: `start` or `stop`
- `duration`: (optional) Number of hours to record (0 or omitted = continuous)

**Example cURL commands:**
```bash
# Start continuous recording
curl "http://192.168.1.100:80/recordingcontrol?action=start&duration=0"

# Start 2-hour recording
curl "http://192.168.1.100:80/recordingcontrol?action=start&duration=2"

# Stop recording
curl "http://192.168.1.100:80/recordingcontrol?action=stop"

# Check status
curl "http://192.168.1.100:80/recordingstatus"
```

### 3. **Video Files**
- **Location**: Saved to SD card at root level
- **Format**: MJPEG (Motion JPEG)
- **Filename Pattern**: `/video_YYYYMMDD_HHMMSS.mjpeg`
- **Example**: `/video_20260301_153045.mjpeg`

## Recording Specifications

### Video Quality
- **Frame Rate**: ~5 FPS (200ms between frames)
- **Format**: MJPEG (Motion JPEG)
- **Resolution**: Depends on camera sensor (SVGA or VGA)
- **Compression**: JPEG encoded frames

### Storage Capacity Estimate
With a 32GB SD card and 5 FPS recording:
- **Bitrate**: ~500-800 kbps (varies by compression quality)
- **Estimated Duration**: ~45-70 hours continuous recording
- **Buffer Management**: Automatic flushing to SD card every frame

## How It Works

### Recording Process
1. Click "START RECORDING" button or use API endpoint
2. The system:
   - Creates a new MJPEG file on the SD card
   - Starts a separate task to capture camera frames
   - Writes each captured frame to the file
   - Flushes data to ensure reliability
3. Recording continues until:
   - You click "STOP RECORDING", or
   - The specified duration expires (if timed mode)
4. File is automatically finalized and saved

### Data Safety
- **Automatic Flushing**: Data is flushed after each frame write
- **File Verification**: System verifies file exists after completion
- **Graceful Shutdown**: Recording task properly terminates and cleans up

## Technical Details

### Key Code Improvements Made
1. **Frame Write Verification**: Confirms all bytes written successfully
2. **File Flushing**: `file.flush()` called after recording stops
3. **File Existence Check**: Verifies file was properly written to SD card
4. **Duration Calculation**: Supports continuous (0 hours) or timed (1-720 hours) recording
5. **Global State Management**: `isRecording` volatile boolean ensures safe state tracking

### API Handler Fixes
- Added `/recordingcontrol` endpoint for start/stop operations
- Added `/recording` endpoint for web UI
- Proper parameter parsing for duration values
- JSON response validation
- CORS headers enabled for cross-origin requests

## Usage Examples

### Web Browser
1. Open: `http://<your-camera-ip>:80/recording`
2. Select recording mode (Continuous or Timed)
3. If Timed, enter duration in hours
4. Click "START RECORDING"
5. Monitor the status display
6. Click "STOP RECORDING" when done

### Programmatic Access
```python
import requests
import time

camera_ip = "192.168.1.100"
base_url = f"http://{camera_ip}:80"

# Start 1-hour recording
response = requests.get(f"{base_url}/recordingcontrol?action=start&duration=1")
print(response.json())  # {"status": "ok"}

# Check status every 5 seconds
for i in range(10):
    time.sleep(5)
    status = requests.get(f"{base_url}/recordingstatus").json()
    print(f"Recording: {status['recording']}")

# Stop recording
response = requests.get(f"{base_url}/recordingcontrol?action=stop")
print(response.json())  # {"status": "ok"}
```

## Troubleshooting

### Recording Won't Start
- Check SD card is properly inserted and formatted (FAT32)
- Verify SD card has available space
- Check camera is properly initialized
- Look at serial monitor for error messages

### Videos Not Saving
- Ensure SD card is properly mounted (check `setup()` logs)
- Verify sufficient disk space (32GB available)
- Check file permissions
- Restart the device and try again

### Low Frame Rate
- This is normal - recording runs at ~5 FPS for reliability
- To increase FPS, modify the `delay(200)` value in `recordVideoTask()`
- Note: Higher FPS = higher SD card wear and larger files

### Status Not Updating
- Refresh the web page
- Wait 2-3 seconds for status to update
- Check browser console for JavaScript errors
- Verify camera responds to normal HTTP requests

## Configuration Options

### Adjusting Frame Rate
Edit `CameraWebServer.ino`, in `recordVideoTask()`:
```cpp
delay(200);  // Change this value
// delay(100) = 10 FPS
// delay(200) = 5 FPS (default)
// delay(500) = 2 FPS
```

### Maximum Recording Hours
Current limit: 720 hours (30 days)
To change, edit the HTML UI in `app_httpd.cpp`:
```javascript
// Find: max="720"
// Change to desired max hours
```

## Performance Notes
- Recording runs on a separate FreeRTOS task (priority 1)
- Main HTTP server remains responsive during recording
- Live streaming (/stream) can be used simultaneously with recording
- ESP32 has sufficient processing power for both operations

## File System Notes
- Videos are MJPEG files compatible with:
  - VLC Media Player
  - FFmpeg
  - Most modern media players
  - Online video converters
- Can be converted to MP4 using FFmpeg:
  ```bash
  ffmpeg -i video_20260301_153045.mjpeg -c:v libx264 -preset ultrafast output.mp4
  ```

## Safety Features Implemented
✅ Volatile flag for thread-safe recording state
✅ Proper task cleanup on recording stop
✅ Memory allocation/deallocation for task parameters
✅ Frame capture error handling
✅ SD card write verification
✅ File flush on completion
✅ Task handle management

## Default Behavior
- Recording is **disabled** by default (`startRecording()` is commented out in setup)
- Uncomment `startRecording(1);` in `setup()` to auto-start 1-hour recording on boot
- Recording state persists in RAM only (not saved across reboots)

## Next Steps / Customization Ideas
1. Add recording start/stop buttons to the main camera interface
2. Add file listing endpoint to view saved videos
3. Add delete/manage files endpoint
4. Implement automatic cleanup when disk is full
5. Add configurable recording parameters (quality, FPS, etc.)
6. Add timestamp overlay on video stream
7. Implement scheduled recording

---

**Last Updated**: March 1, 2026
**Tested with**: ESP32 with 32GB SD card, SVGA resolution
