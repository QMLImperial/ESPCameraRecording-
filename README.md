# ESPCameraRecording-
IPADDRESS:80/recording


ffmpeg -f mjpeg -r 5 -i video_YYYYMMDD_HHMMSS.mjpeg -c:v libx264 -preset veryfast -crf 23 out.mp4


$env:Path += ";C:\ffmpeg\bin
