
---

# sam2_realtime

Run Segment Anything Model 2 (SAM2) on a **live video stream** with new updates and fixes for enhanced functionality and usability.

---

## News

- **15/11/2024**:  
  - Added support for **streaming with Flask on Windows to WSL2**.
  - Updated **SAM2 to SAM2.1** and resolved issues with `pip install`.
  - Fixed `object_score_logits` error for seamless execution.
  - Simplified `camera_predict_demo.py` to enable execution by mouse click.

---

## Demos
<div align="center">
  <video src="realtime.mp4" width="880" controls>
    Your browser does not support the video tag.
  </video>
</div>


## Getting Started

### **System Requirements**
- **Python version**: Python 3.11
- **CUDA**:
  - NVIDIA-SMI: 12.6
  - NVCC Version: CUDA 12.3
  - Torch Version: 2.4.1 (CUDA 12.1)

---

### **Installation**

Clone the repository and install the required packages:
```bash
pip install -e .
```

---

### **Download Checkpoint**
Download the required model checkpoint:

```bash
cd checkpoints
./download_ckpts.sh
```

---

### **Usage**

#### Camera Prediction

Run SAM2 real-time prediction on a live camera feed:
```python
import torch
from sam2.build_sam import build_sam2_camera_predictor

sam2_checkpoint = "../checkpoints/sam2.1_hiera_small.pt"
model_cfg = "configs/sam2.1/sam2.1_hiera_s.yaml"

predictor = build_sam2_camera_predictor(model_cfg, sam2_checkpoint, device=device)

cap = cv2.VideoCapture(0)  # Use 0 for default camera

if_init = False

with torch.inference_mode(), torch.autocast("cuda", dtype=torch.bfloat16):
    while True:
        ret, frame = cap.read()
        if not ret:
            break
        width, height = frame.shape[:2][::-1]

        if not if_init:
            predictor.load_first_frame(frame)
            if_init = True
            _, out_obj_ids, out_mask_logits = predictor.add_new_prompt("Your Prompt Here")

        else:
            out_obj_ids, out_mask_logits = predictor.track(frame)
            # Add custom processing logic here
```

---

### **Updates and Fixes**

#### 1. **Streaming with Flask on Windows to WSL2**
   - You can now stream video from Windows using Flask and consume it in WSL2.  
     Refer to the detailed guide in [streaming.md](./streaming.md) for setup instructions.

#### 2. **Fixed SAM2 to SAM2.1 Update Issue**
   - Resolved the `pip install` problem after updating to SAM2.1. If you face any issues, reinstall dependencies using:
     ```bash
     pip install --upgrade .
     ```

#### 3. **Resolved `object_score_logits` Error**
   - Corrected logic to handle missing `object_score_logits` in new SAM2 versions. Ensure you're using the latest repository updates.

#### 4. **Simplified Camera Prediction Demo**
   - `camera_predict_demo.py` is now executable with a simple double-click. Use this script to run real-time segmentation directly.

---

### **References**

- SAM2 Repository: [https://github.com/facebookresearch/segment-anything-2](https://github.com/facebookresearch/segment-anything-2)
- Real-Time Implementation: [https://github.com/Gy920/segment-anything-2-real-time](https://github.com/Gy920/segment-anything-2-real-time)

---

### **Contributing**
If you encounter any issues or have suggestions for improvement, feel free to open an issue or submit a pull request.

---
