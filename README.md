import cv2

# Start webcam capture
cap = cv2.VideoCapture(0)

# Check if webcam opened successfully
if not cap.isOpened():
    print("Error: Could not open webcam.")
    exit()

while True:
    ret, frame = cap.read()
    if not ret:
        print("Error: Failed to read frame.")
        break

    # Show the frame
    cv2.imshow('Webcam Feed - Press q to Quit', frame)

    # Wait for key press
    if cv2.waitKey(1) & 0xFF == ord('q'):
        print("Quitting...")
        break

# Release resources
cap.release()
cv2.destroyAllWindows()
import cv2
import numpy as np
import math

def count_fingers(contour, drawing):
    hull = cv2.convexHull(contour, returnPoints=False)
    if len(hull) > 3:
        defects = cv2.convexityDefects(contour, hull)
        if defects is not None:
            finger_count = 0
            for i in range(defects.shape[0]):
                s, e, f, d = defects[i, 0]
                start = tuple(contour[s][0])
                end = tuple(contour[e][0])
                far = tuple(contour[f][0])

                # Calculate angle
                a = math.dist(end, start)
                b = math.dist(far, start)
                c = math.dist(end, far)
                angle = math.acos((b**2 + c**2 - a**2) / (2*b*c)) * 57

                # Filter based on angle and depth
                if angle <= 90 and d > 10000:
                    finger_count += 1
                    cv2.circle(drawing, far, 8, [255, 0, 0], -1)

            return finger_count + 1
    return 0
# Start webcam capture
cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    frame = cv2.flip(frame, 1)
    roi = frame[100:400, 100:400]
    cv2.rectangle(frame, (100, 100), (400, 400), (0, 255, 0), 2)

    gray = cv2.cvtColor(roi, cv2.COLOR_BGR2GRAY)
    blur = cv2.GaussianBlur(gray, (35, 35), 0)
    _, thresh = cv2.threshold(blur, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)

    contours, _ = cv2.findContours(thresh, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
    if contours:
        max_contour = max(contours, key=cv2.contourArea)
        drawing = np.zeros(roi.shape, np.uint8)
        cv2.drawContours(drawing, [max_contour], -1, (0, 255, 0), 2)

        fingers = count_fingers(max_contour, drawing)

        # Map finger count to gesture
        gestures = {
            0: "Fist",
            1: "One",
            2: "Peace ✌️",
            3: "Three",
            4: "Four",
            5: "Open Hand 🖐️",
            6: "Swag 🤘"
        }
        gesture_name = gestures.get(fingers, "Unknown")
        cv2.putText(frame, f"Gesture: {gesture_name}", (50, 50), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 255), 2)

        frame[100:400, 100:400] = drawing

    cv2.imshow("Hand Gesture Recognition", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
import cv2

# Start webcam
cap = cv2.VideoCapture(0)

# Check if webcam opened successfully
if not cap.isOpened():
    print("Error: Could not open webcam.")
    exit()

try:
    while True:
        ret, frame = cap.read()
        if not ret:
            print("Error: Failed to read frame.")
            break

        cv2.imshow("Webcam Feed - Press 'q' to Quit", frame)

        # Exit on pressing 'q'
        if cv2.waitKey(1) & 0xFF == ord('q'):
            print("Quitting...")
            break

except KeyboardInterrupt:
    print("Interrupted manually.")
    
