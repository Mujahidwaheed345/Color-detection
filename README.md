import cv2
from PIL import Image
import sys
import numpy as np

# Try to import get_limits from util.py
try:
    from util import get_limits  # Ensure util.py is in the same directory
except ImportError:
    print("❌ Error: 'util.py' not found. Ensure it's in the same directory.")
    sys.exit(1)

# 🎨 Define Colors in BGR (Except Black & White, which need special handling)
BLUE = [255, 0, 0]
YELLOW = [0, 255, 255]
GREEN = [0, 255, 0]
PINK = [255, 0, 255]  # Magenta / Pink
BROWN = [42, 42, 165]  # Brown in BGR (Dark Orange)

# 🎥 Function to find a working camera index
def find_camera():
    for index in range(5):  # Try indices 0-4
        cap = cv2.VideoCapture(index, cv2.CAP_DSHOW)  # Use DirectShow on Windows
        if cap.isOpened():
            print(f"✅ Camera found at index {index}")
            cap.release()
            return index
    print("❌ Error: No available camera found.")
    sys.exit(1)

# 🔍 Find working camera index
camera_index = find_camera()
cap = cv2.VideoCapture(camera_index, cv2.CAP_DSHOW)

if not cap.isOpened():
    print("❌ Error: Could not open camera.")
    sys.exit(1)

# 📹 Video Capture Loop
while True:
    ret, frame = cap.read()
    if not ret or frame is None:
        print("❌ Error: Failed to capture frame.")
        break  

    # Convert frame to HSV
    hsvImage = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

    # Get color limits
    lower_blue, upper_blue = get_limits(color=BLUE)
    lower_yellow, upper_yellow = get_limits(color=YELLOW)
    lower_green, upper_green = get_limits(color=GREEN)
    lower_pink, upper_pink = get_limits(color=PINK)
    lower_brown, upper_brown = get_limits(color=BROWN)

    # **Custom Black & White Detection**
    # White: High brightness, low saturation
    lower_white = np.array([0, 0, 200])  # Bright
    upper_white = np.array([180, 50, 255])  

    # Black: Low brightness
    lower_black = np.array([0, 0, 0])
    upper_black = np.array([180, 255, 50])  # Dark

    # Create binary masks
    masks = {
        "White": (cv2.inRange(hsvImage, lower_white, upper_white), (200, 200, 200)),  # Gray box
        "Black": (cv2.inRange(hsvImage, lower_black, upper_black), (255, 255, 255)),  # White box for visibility
        "Blue": (cv2.inRange(hsvImage, lower_blue, upper_blue), (255, 0, 0)),  # Blue box
        "Yellow": (cv2.inRange(hsvImage, lower_yellow, upper_yellow), (0, 255, 255)),  # Yellow box
        "Green": (cv2.inRange(hsvImage, lower_green, upper_green), (0, 255, 0)),  # Green box
        "Pink": (cv2.inRange(hsvImage, lower_pink, upper_pink), (255, 0, 255)),  # Magenta box
        "Brown": (cv2.inRange(hsvImage, lower_brown, upper_brown), (19, 69, 139)),  # Brown box (Darker)
    }

    # Process each color mask
    for color_name, (mask, box_color) in masks.items():
        mask_pil = Image.fromarray(mask)
        bbox = mask_pil.getbbox()

        if bbox:
            x1, y1, x2, y2 = bbox
            cv2.rectangle(frame, (x1, y1), (x2, y2), box_color, 5)
            cv2.putText(frame, color_name, (x1, y1 - 10), cv2.FONT_HERSHEY_SIMPLEX, 0.7, box_color, 2)

    # Show the frame
    cv2.imshow('Color Detection', frame)

    # Press 'q' to exit
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# 🛑 Release camera & destroy windows
cap.release()
cv2.destroyAllWindows()
