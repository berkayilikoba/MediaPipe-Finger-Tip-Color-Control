import cv2 as cv
import mediapipe as mp
import time
import serial

# Initialize MediaPipe hands solution
mp_hands = mp.solutions.hands
mp_drawing = mp.solutions.drawing_utils
hands = mp_hands.Hands(min_detection_confidence=0.5, min_tracking_confidence=0.5)

# Set up the serial communication with Arduino
arduino = serial.Serial(port='COM5', baudrate=9600, timeout=1)
time.sleep(2)  # Wait for 2 seconds to allow Arduino to initialize

# Initialize the camera
cap = cv.VideoCapture(1)  # Start the camera
if not cap.isOpened():
    print("Camera could not be opened!")
    exit()

# Square size and center position for the squares
square_size = 100
center_y = 250  # Fixed vertical center for the squares

# Frame width (640 by default)
frame_width = 640

# X coordinates for the squares
left_x = 50
center_x = (frame_width - square_size) // 2
right_x = frame_width - square_size - 50

# Coordinates for the Red, Green, and Blue squares
red_x1, red_y1 = left_x, center_y - square_size // 2
red_x2, red_y2 = left_x + square_size, center_y + square_size // 2

green_x1, green_y1 = center_x, center_y - square_size // 2
green_x2, green_y2 = center_x + square_size, center_y + square_size // 2

blue_x1, blue_y1 = right_x, center_y - square_size // 2
blue_x2, blue_y2 = right_x + square_size, center_y + square_size // 2

# Timer variables
last_time = time.time()
current_color = None

# Function to check if a point (finger tip) is inside a rectangle
def is_inside(x, y, x1, y1, x2, y2):
    """Check if the point (x, y) is inside the rectangle defined by (x1, y1) and (x2, y2)."""
    return x1 <= x <= x2 and y1 <= y <= y2

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        print("Failed to capture frame, exiting.")
        break

    frame_height, frame_width, _ = frame.shape  # Get the current frame width and height
    rgb_frame = cv.cvtColor(frame, cv.COLOR_BGR2RGB)
    result = hands.process(rgb_frame)

    finger_inside = None  # Variable to track which square the finger is inside

    if result.multi_hand_landmarks:
        for hand_landmarks in result.multi_hand_landmarks:
            mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)

            # Get the coordinates of the index finger tip (landmark 8)
            index_finger_tip = hand_landmarks.landmark[8]
            finger_x = int(index_finger_tip.x * frame_width)
            finger_y = int(index_finger_tip.y * frame_height)

            # Check which square the finger is inside
            if is_inside(finger_x, finger_y, red_x1, red_y1, red_x2, red_y2):
                finger_inside = "red"
            elif is_inside(finger_x, finger_y, green_x1, green_y1, green_x2, green_y2):
                finger_inside = "green"
            elif is_inside(finger_x, finger_y, blue_x1, blue_y1, blue_x2, blue_y2):
                finger_inside = "blue"

    # If the finger stays inside a square for more than 1 second, send the corresponding color to Arduino
    if finger_inside == current_color and time.time() - last_time >= 1:
        arduino.write(f"{finger_inside}\n".encode())  # Send the color to Arduino
        print(f"Finger stayed in the {finger_inside} square for 1 second.")
        current_color = None  # Reset to avoid sending multiple messages for the same square
    elif finger_inside != current_color:
        current_color = finger_inside
        last_time = time.time()  # Reset the timer

    # Draw the Red, Green, and Blue squares on the frame
    cv.rectangle(frame, (red_x1, red_y1), (red_x2, red_y2), (0, 0, 255), -1)  # Red square
    cv.rectangle(frame, (green_x1, green_y1), (green_x2, green_y2), (0, 255, 0), -1)  # Green square
    cv.rectangle(frame, (blue_x1, blue_y1), (blue_x2, blue_y2), (255, 0, 0), -1)  # Blue square

    # Display the frame with squares
    cv.imshow("Centered Squares", frame)
    
    key = cv.waitKey(1)
    if key == ord("q"):
        break

# Release the camera and close all OpenCV windows
cap.release()
cv.destroyAllWindows()
