# AI Gym Trainer · Voice Assisted Pose & Rep Tracker

AI Gym Trainer is a voice controlled workout assistant that uses computer vision to track your form, count reps, and give you a quick summary of what you did at the end of the session.

Under the hood it uses MediaPipe Pose and OpenCV to detect keypoints on your body, computes joint angles to understand each exercise, and wraps everything in a friendly voice assistant called **Chris** who talks you through the workout.

This project is based on an academic paper on a **voice assisted fitness coach with body pose recognition**.

---

## Features

- 🎙 **Voice assistant interface**
  - Hands free control using your mic
  - Wake it with exercise names like “squats,” “left bicep curl,” or “push ups”
  - Say “quit” to end the session and get a spoken summary

- 🧍‍♂️ **Real time pose estimation**
  - Uses MediaPipe Pose through your webcam
  - Tracks key joints needed for each exercise
  - Calculates angles to determine movement phases

- 🔢 **Automatic rep counting**
  - Detects up and down phases based on joint angles
  - Robust to small jitters using “stage” switches (up / down)
  - Counts per exercise and stores them across the session

- 📊 **End of workout summary**
  - Speaks out reps per exercise
  - Prints a text summary in the console
  - Generates a bar chart of all exercises and reps for that session
  - Saves the figure with a timestamped filename

- 🏋️ Supported exercises (in this version)
  - Full Bicep Curl
  - Left Bicep Curl
  - Right Bicep Curl
  - Squats
  - Lunges
  - Push Ups
  - Lower Abdominal Crunches
  - High Knees
  - Jumping Jacks
  - Mountain Climbers

---

## Tech Stack

- **Language**: Python
- **Computer Vision**: OpenCV, MediaPipe Pose
- **Math & Arrays**: NumPy
- **Visualization**: Matplotlib
- **Voice & Audio**:
  - `pyttsx3` for text to speech (configured with `sapi5` voice engine)
  - `speech_recognition` for microphone input and Google Speech API
- **Other**: Standard Python libraries like `time`, `datetime`, `random`, etc.

> Note: `pyttsx3` is currently initialized with the Windows `sapi5` engine. On macOS or Linux you will need to adjust the initialization.

---

## How It Works

### 1. Pose detection and angle calculation

Each exercise function (for example `squat()`, `fullbicep()`, `pushup()`, etc.):

1. Opens the webcam using OpenCV (`cv2.VideoCapture(0)`).
2. Runs a MediaPipe Pose model on each frame.
3. Extracts relevant landmarks such as:
   - Shoulder, elbow, wrist for curls
   - Hip, knee, ankle for squats and lunges
4. Calls a shared helper like `calculate_angle(a, b, c)` to compute joint angles.

Based on the angle thresholds, the code infers whether the user is in the “up” or “down” position and toggles a `stage` variable.

When the stage switches from “down” to “up” (or vice versa, depending on exercise), one rep is counted.

### 2. Exercise specific logic

Each movement has its own function:

- `fullbicep()`
- `leftbicep()`
- `rightbicep()`
- `squat()`
- `lunge()`
- `pushup()`
- `abcrunch()`
- `jumpingjacks()`
- `highknees()`
- `mountainclimb()`

Inside each:

- The live video is shown with:
  - Pose skeleton drawn on top via `mp_drawing.draw_landmarks`
  - On screen overlays for:
    - Current rep count
    - Current stage (e.g. “UP”, “DOWN”)
- Pressing **`q`** closes that exercise window.
- The function returns the total `counter` for that exercise.
- Chris then says something like  
  `“Good job! You completed 12 reps of the left bicep curl.”`

### 3. Voice assistant “Chris”

Core helpers:

- `take()`  
  Listens through the microphone, uses `speech_recognition` and Google’s speech API to turn your speech into text, and returns the recognized command.

- `speak(text)`  
  Uses `pyttsx3` to speak the passed text.

- `wishMe()`  
  Greets the user based on the time of day and introduces Chris as your personal gym trainer.

The main loop:

```python
if __name__ == '__main__':
    wishMe()
    while True:
        query = take().lower()

        if 'squat' in query or 'squats' in query:
            # Start squat workout
        elif 'full bicep' in query:
            # Start full bicep curls
        ...
        elif 'the time' in query:
            # Tells you the current time
        elif 'quit' in query:
            # Reads out summary, plots stats, and exits
