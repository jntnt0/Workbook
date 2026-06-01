"""
Compare two screen regions using HOG feature descriptors to assess
visual similarity in formatting or font characteristics.

Captures a screenshot, lets the user select two regions of interest (ROIs),
computes HOG descriptors for each, and reports the Euclidean distance between them.

Usage:
    python hog_region_compare.py

Requirements:
    - opencv-python
    - numpy
    - pyautogui
    - scikit-image
    - scipy
"""

#!/usr/bin/env python3
import cv2
import numpy as np
import pyautogui
from skimage.feature import hog
from scipy.spatial.distance import euclidean

def capture_screenshot():
    # Capture a full-screen screenshot using pyautogui and convert it to a BGR numpy array for OpenCV.
    screenshot = pyautogui.screenshot()
    img = cv2.cvtColor(np.array(screenshot), cv2.COLOR_RGB2BGR)
    return img

def select_roi(image, window_name="Select ROI"):
    # Display the image and let the user select a ROI.
    # cv2.selectROI returns (x, y, w, h)
    roi = cv2.selectROI(window_name, image, False, False)
    cv2.destroyWindow(window_name)
    x, y, w, h = roi
    if w and h:
        return image[y:y+h, x:x+w]
    else:
        return None

def compute_hog_descriptor(image_gray):
    # Compute a HOG descriptor for the grayscale image.
    # You can tweak parameters like orientations, pixels_per_cell, and cells_per_block
    hog_desc, _ = hog(image_gray,
                      orientations=9,
                      pixels_per_cell=(8, 8),
                      cells_per_block=(2, 2),
                      block_norm='L2-Hys',
                      visualize=True,
                      feature_vector=True)
    return hog_desc

def main():
    print("Capturing the screen in 3 seconds. Get ready...")
    cv2.waitKey(3000)  # Wait 3 seconds
    screenshot = capture_screenshot()

    # Let the user select the first ROI
    cv2.imshow("Full Screenshot - Press any key to continue", screenshot)
    cv2.waitKey(500)
    print("Select the first text region.")
    roi1 = select_roi(screenshot, "Select ROI #1")
    if roi1 is None:
        print("No ROI selected for the first region.")
        return

    # Let the user select the second ROI
    cv2.imshow("Full Screenshot - Press any key to continue", screenshot)
    cv2.waitKey(500)
    print("Select the second text region.")
    roi2 = select_roi(screenshot, "Select ROI #2")
    if roi2 is None:
        print("No ROI selected for the second region.")
        return

    # Convert ROIs to grayscale for feature extraction
    roi1_gray = cv2.cvtColor(roi1, cv2.COLOR_BGR2GRAY)
    roi2_gray = cv2.cvtColor(roi2, cv2.COLOR_BGR2GRAY)

    # Compute HOG descriptors
    hog1 = compute_hog_descriptor(roi1_gray)
    hog2 = compute_hog_descriptor(roi2_gray)

    # Compute Euclidean distance between the HOG descriptors
    distance = euclidean(hog1, hog2)
    print("Euclidean distance between HOG descriptors:", distance)

    # Define a threshold value that indicates similarity.
    # This threshold is arbitrary and may require tuning.
    similarity_threshold = 0.5  # Example value; adjust as needed.

    if distance < similarity_threshold:
        print("The two text sections appear to use similar formatting/font characteristics.")
    else:
        print("The two text sections appear to be different in formatting/font characteristics.")

if __name__ == '__main__':
    main()
