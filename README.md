# robot-projects
Arduino-based robots with camera sensing, voice and gesture control
What this robot does
— overviewThis robot has four major capabilities working simultaneously: it sees faces, listens to your voice, reads your hand gestures, and drives on 4 wheels. These aren't separate devices — they're all coordinated by one central brain running on a Raspberry Pi. Let's go through each system one by one.
System 1 — Face RecognitionThe ESP32-CAM captures a live video stream over WiFi. The Raspberry Pi reads that stream using OpenCV, shrinks each frame to 25% size for speed, then runs the face_recognition library to find and identify faces. You pre-load photos of known people into a known_faces/ folder — the robot encodes each face mathematically at startup. When it sees a match live, it greets that person by name using the text-to-speech engine.
python:
   # Shrink frame 4x for speed
small_frame = cv2.resize(frame, (0, 0), fx=0.25, fy=0.25)

# Find faces and encode them
face_locations = face_recognition.face_locations(rgb_small)
face_encodings = face_recognition.face_encodings(rgb_small, face_locations)

# Compare against your known people
matches = face_recognition.compare_faces(known_face_encodings, face_encoding)
System 2 — Voice InteractionThe USB microphone continuously listens. When speech is detected, it sends the audio to Google's Speech Recognition API (needs internet), gets back text, and then your code checks that text against two dictionaries: one for movement commands and one for conversation phrases. The robot either moves a motor or speaks a reply.
python:
   # Listen for speech (waits up to 5 seconds)
audio = recognizer.listen(source, timeout=5, phrase_time_limit=6)

# Convert to text via Google
text = recognizer.recognize_google(audio)

# Check if it's a motor command
for phrase, cmd in VOICE_COMMANDS.items():
    if phrase in text.lower():
        send_motor_command(cmd)   # e.g. "F" = forward
        time.sleep(1.5)
        send_motor_command("S")   # auto-stop after 1.5 seconds
System 4 — 4-Wheel Motor ControlThe Arduino Mega receives single-character commands over USB Serial from the Pi and drives two L298N H-bridge motor driver modules — one for the left pair of wheels, one for the right pair. PWM on pins ENA/ENB controls speed; IN1–IN4 control direction.
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 0.25s;
    animation-timing-function: cubic-bezier(0.19, 1, 0.22, 1);
  }
VvisualizeVvisualize show_widget
cpp:
  void turnLeft() {
  // Right side forward, Left side backward = spins left on the spot
  digitalWrite(IN1, LOW);  digitalWrite(IN2, HIGH);  // left: reverse
  digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);   // right: forward
}
cpp:
   void curveForwardLeft() {
  analogWrite(ENA, motorSpeed / 2);  // Left wheels slower
  analogWrite(ENB, motorSpeed);       // Right wheels full speed
  // Both directions forward → gentle left curve
}
How all 4 systems run together
The Raspberry Pi runs all three perception systems (face, voice, gesture) as separate Python threads simultaneously. None of them block each other. They all call the same send_motor_command() function when action is needed, which writes to the Arduino over Serial.
python:
    threads = [
    threading.Thread(target=face_recognition_thread, daemon=True),
    threading.Thread(target=voice_command_thread,    daemon=True),
    threading.Thread(target=gesture_control_thread,  daemon=True),
]
for t in threads:
    t.start()
    
