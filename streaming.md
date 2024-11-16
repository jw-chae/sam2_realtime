
---

# Streaming from Windows to WSL2

This guide walks you through how to stream video from a **Windows** system to **WSL2** using Flask and OpenCV. The video feed from a webcam will be streamed to WSL2, where you can process it in real-time.

## Prerequisites

- **Windows** machine with **WSL2** enabled.
- **Python 3.11** or newer installed in both Windows and WSL2 environments.
- **Flask** and **OpenCV** installed on Windows.
- **requests** and **OpenCV** installed on WSL2.

---

## Step 1: Install Required Libraries

### On Windows (Flask server):

Install **Flask** and **OpenCV** on Windows:
```bash
pip install flask opencv-python
```

### On WSL2 (client side):

Install **requests** and **OpenCV** in your WSL2 environment:
```bash
pip install opencv-python requests
```

---

## Step 2: Setting Up Flask Server on Windows

In this step, we will set up a simple Flask server that streams video from the webcam.

### Create a Python script `flask_webcam_stream.py` on Windows:
```python
from flask import Flask, Response
import cv2

app = Flask(__name__)

# Open the webcam (0 for the default camera)
camera = cv2.VideoCapture(0)

def generate_frames():
    while True:
        success, frame = camera.read()
        if not success:
            break
        else:
            # Encode the frame as JPEG
            _, buffer = cv2.imencode('.jpg', frame)
            frame = buffer.tobytes()
            # Yield the frame as a multipart response for streaming
            yield (b'--frame\r\n'
                   b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')

@app.route('/video_feed')
def video_feed():
    return Response(generate_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

- **Explanation**:
  - We open the webcam using `cv2.VideoCapture(0)`.
  - The `generate_frames()` function continuously captures frames from the webcam, encodes them as JPEG, and sends them as a multipart response (which is suitable for video streaming).
  - The Flask server runs on `0.0.0.0:5000`, making it accessible over the network.

### Run the Flask Server:
In the Windows terminal, run the Flask server:
```bash
python flask_webcam_stream.py
```

---

## Step 3: Accessing the Video Stream from WSL2

Now that the Flask server is running on Windows, we can access the video stream from WSL2. In WSL2, we'll use OpenCV and `requests` to get the video stream and display it.

### Create a Python script `wsl_streaming.py` on WSL2:
```python
import cv2
import requests
import numpy as np

url = "http://<Windows_IP>:5000/video_feed"  # Replace <Windows_IP> with your Windows machine's IP

# Make a request to the Flask server to start receiving video stream
cap = requests.get(url, stream=True)
bytes_data = b''

for chunk in cap.iter_content(chunk_size=1024):
    bytes_data += chunk
    a = bytes_data.find(b'\xff\xd8')  # JPEG start marker
    b = bytes_data.find(b'\xff\xd9')  # JPEG end marker
    if a != -1 and b != -1:
        jpg = bytes_data[a:b+2]
        bytes_data = bytes_data[b+2:]

        # Decode the JPEG data into an OpenCV frame
        frame = cv2.imdecode(np.frombuffer(jpg, dtype=np.uint8), cv2.IMREAD_COLOR)

        # Display the frame
        cv2.imshow("WSL Stream", frame)
        if cv2.waitKey(1) == 27:  # ESC key to exit
            break

cv2.destroyAllWindows()
```

- **Explanation**:
  - We create a connection to the Flask server on Windows by sending a `GET` request to `http://<Windows_IP>:5000/video_feed`.
  - The video feed is received as a stream of JPEG images, which are decoded using `cv2.imdecode()` and displayed using `cv2.imshow()`.

### Run the WSL2 Client:
In your WSL2 terminal, run the Python script:
```bash
python wsl_streaming.py
```

You should now see the live video stream from the Windows webcam, displayed within your WSL2 environment!

---

## Step 4: Troubleshooting

- **Windows Firewall**:  
  If you're unable to access the stream, make sure that the Flask server is accessible through the firewall. You might need to allow port `5000` in the **Windows Defender Firewall** settings.
  
- **Finding the Windows IP**:
  To find the Windows IP address, run the following command in Command Prompt (CMD) or PowerShell:
  ```bash
  ipconfig
  ```

  Look for the **IPv4 Address** under your network adapter settings (e.g., `192.168.x.x`).

- **WSL2 Networking**:  
  In WSL2, Windows is accessible through a specific IP. If the connection to `http://<Windows_IP>:5000` fails, use the default WSL2 IP address `172.19.208.1` or find the dynamic IP assigned to your WSL2 instance.

---

## Additional Features

- **Optional: Streaming from Other Sources**  
  You can modify the script to stream video from a file or another camera by changing the `cv2.VideoCapture` source in the Flask server script:
  ```python
  # Use a video file or an additional camera
  cap = cv2.VideoCapture('path_to_video.mp4')  # Or use another camera index (e.g., 1 for secondary camera)
  ```

- **Optional: Adjust Stream Quality**  
  Adjust the quality of the stream by changing the `cv2.imencode()` compression parameters:
  ```python
  _, buffer = cv2.imencode('.jpg', frame, [int(cv2.IMWRITE_JPEG_QUALITY), 80])  # 80% quality
  ```

---
