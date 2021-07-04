---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: 드론
---
드론 관련 

1. Raspberry Pi 
2. flight controller + Ardupilot
3. Camera module

Connection between RPi and PC through UDP

Face detection, object recognition, flight planning on server PC

1. UDP connection

2. Face detection on server PC using [face_recognition](https://github.com/ageitgey/face_recognition)

```python

import numpy as np
import cv2
import face_recognition
while(True):
	#incoming RPi camera stream as PiRGBArray
	screen = "PiRGBArray"
  
	#face location on screen, format is: (top, right, bottom, left)
	face_locations = face_recognition.face_locations(screen)

	#center of faces 
	center_all = []
	for faces in face_locations:
		center_face = (int(round(faces[1]+faces[3])/2), int(round(faces[2]+faces[0])/2))
		center_all.append(center_face)
		screen = cv2.circle(screen, center_face, 5, (0,0,255), -1)

	if len(center_all) >= 1:
		center_all = tuple(map(lambda x: int(round(sum(x)/len(x))), zip(*center_all)))
		screen = cv2.circle(screen, center_all, 5, (255,0,0), -1)

	cv2.imshow('window')

	if cv2.waitKey(25) & 0xFF == ord('q'):
		cv2.destroyAllWindows()
		print(center_all)
		break

```