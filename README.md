RUNNING PATTERN
=================

import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

pins = [5, 6, 13, 19, 26]
for pin in pins:
    GPIO.setup(pin, GPIO.OUT)

try:
    for pin in pins:
        GPIO.output(pin, GPIO.HIGH)
        time.sleep(0.5)
        GPIO.output(pin, GPIO.LOW)
        time.sleep(0.5)
finally:
    for pin in pins:
        GPIO.output(pin, GPIO.LOW)
    GPIO.cleanup()

=============
CENTER DASH
=============

import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

pins = [5, 6, 13, 19, 26]

for pin in pins:
    GPIO.setup(pin, GPIO.OUT)

try:
    for num in range((len(pins) // 2) + 1):
        GPIO.output(pins[num], GPIO.HIGH)
        GPIO.output(pins[len(pins) - num - 1], GPIO.HIGH)
        time.sleep(0.5)
        GPIO.output(pins[num], GPIO.LOW)
        GPIO.output(pins[len(pins) - num - 1], GPIO.LOW)
        time.sleep(0.5)
finally:
    GPIO.cleanup()

================
SEVEN SEGMENT 
================

import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)

segments = (17, 27, 22, 23, 24, 25, 5, 6) 
for segment in segments:
    GPIO.setup(segment, GPIO.OUT)
    GPIO.output(segment, 1)

digits = (16, 20, 21, 12)  
for digit in digits:
    GPIO.setup(digit, GPIO.OUT)
    GPIO.output(digit, 1)


num = {
    ' ': (0,0,0,0,0,0,0),
    '0': (1,1,1,1,1,1,0),
    '1': (0,1,1,0,0,0,0),
    '2': (1,1,0,1,1,0,1),
    '3': (1,1,1,1,0,0,1),
    '4': (0,1,1,0,0,1,1),
    '5': (1,0,1,1,0,1,1),
    '6': (1,0,1,1,1,1,1),
    '7': (1,1,1,0,0,0,0),
    '8': (1,1,1,1,1,1,1),
    '9': (1,1,1,1,0,1,1)
}

try:
    while True:
        # Show current time in HHMM
        n = time.strftime("%H%M")
        for digit in range(4):
            for loop in range(7):  # loop through a–g
                GPIO.output(segments[loop], not num[n[digit]][loop])  # invert for common anode
            GPIO.output(digits[digit], 0)  # activate current digit
            time.sleep(0.003)
            GPIO.output(digits[digit], 1)  # deactivate digit
except KeyboardInterrupt:
    GPIO.cleanup()

=================
CAMERA 
=================

from time import sleep
from picamera import PiCamera

camera = PiCamera()
camera.resolution = (1280, 720)

camera.start_preview()
sleep(5)  # Give time for camera to adjust lighting/focus
camera.capture('/home/pi/Pictures/group12.jpg')
camera.stop_preview()

================
OSCILLOSCOPE 
=================

import time
import matplotlib.pyplot as plt
from drawnow import drawnow
import Adafruit_ADS1x15

# Initialize ADC
adc = Adafruit_ADS1x15.ADS1115()
GAIN = 1

val = []
cnt = 0

plt.ion()  

adc.start_adc(0, gain=GAIN)
print("Reading values from ADC Channel 0... Press Ctrl+C to stop.")

def makeFig():
    plt.ylim(-18000, 18000)
    plt.title('Real-Time Oscilloscope View')
    plt.grid(True)
    plt.ylabel('ADC Output')
    plt.plot(val, 'ro-', label='Channel 0')
    plt.legend(loc='lower right')

try:
    while True:
        value = adc.get_last_result()
        print(f'Channel 0: {value}')
        val.append(int(value))
        drawnow(makeFig)
        plt.pause(0.05)  
        cnt += 1
        if cnt > 50:
            val.pop(0)
        time.sleep(0.1)
except KeyboardInterrupt:
    print("\nStopping ADC reading...")
finally:
    adc.stop_adc()
    print("ADC stopped safely.")

=============
WRITE 
=============

#!/usr/bin/env python
import RPi.GPIO as GPIO
from mfrc522 import SimpleMFRC522

reader=SimpleMFRC522()
GPIO.setwarnings(False)
try: 
    text=input('New data:')
    print("Now place your tag to write")
    reader.write(text)
    print("Written")
finally:
    GPIO.cleanup()

==============
READ
==============

#!/usr/bin/env python
import RPi.GPIO as GPIO
from mfrc522 import SimpleMFRC522

reader = SimpleMFRC522()

try:
    print("Place your RFID tag near the reader...")
    id, text = reader.read()
    print("ID:", id)
    print("Text:", text)
finally:
    GPIO.cleanup()


=============
TELEGRAM 
=============

import sys
import time
import telepot
import RPi.GPIO as GPIO

# -------------------------
# Setup GPIO
# -------------------------
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)
LED_PIN = 20
GPIO.setup(LED_PIN, GPIO.OUT)

# -------------------------
# LED Control Functions
# -------------------------
def on(pin):
    GPIO.output(pin, GPIO.HIGH)
    return '💡 LED ON'

def off(pin):
    GPIO.output(pin, GPIO.LOW)
    return '💡 LED OFF'

# -------------------------
# Telegram Bot Handler
# -------------------------
def handle(msg):
    chat_id = msg['chat']['id']
    command = msg['text'].strip().lower()

    print(f"Got command: {command}")

    if command == 'on':
        bot.sendMessage(chat_id, on(LED_PIN))
    elif command == 'off':
        bot.sendMessage(chat_id, off(LED_PIN))
    elif command == 'status':
        state = GPIO.input(LED_PIN)
        bot.sendMessage(chat_id, f"LED is {'ON' if state else 'OFF'}")
    else:
        bot.sendMessage(chat_id, "❌ Unknown command.\nTry: 'on', 'off', or 'status'.")

# -------------------------
# Initialize Telegram Bot
# -------------------------
TOKEN = 'YOUR_TELEGRAM_BOT_TOKEN_HERE'  # 👈 Replace this with your bot token
bot = telepot.Bot(TOKEN)
bot.message_loop(handle)

print("🤖 Bot is listening... Send 'on', 'off', or 'status' to control the LED.")

# -------------------------
# Keep Running
# -------------------------
try:
    while True:
        time.sleep(10)

except KeyboardInterrupt:
    print("\nProgram interrupted by user.")
    GPIO.cleanup()
    sys.exit(0)

except Exception as e:
    print(f"⚠️ Error: {e}")
    GPIO.cleanup()
    sys.exit(1)

