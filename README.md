# DoseEase
import time
import RPi.GPIO as GPIO
from datetime import datetime
from twilio.rest import Client

# GPIO Setup
SERVO_PIN = 18
GPIO.setmode(GPIO.BCM)
GPIO.setup(SERVO_PIN, GPIO.OUT)
servo = GPIO.PWM(SERVO_PIN, 50)

# Twilio Setup for SMS Alerts
TWILIO_SID = 'your_twilio_sid'
TWILIO_AUTH_TOKEN = 'your_twilio_auth_token'
TWILIO_PHONE = 'your_twilio_phone_number'
USER_PHONE = 'user_phone_number'

twilio_client = Client(TWILIO_SID, TWILIO_AUTH_TOKEN)

# Medication Schedule (time in 24-hour format)
med_schedule = {
    "morning": "08:00",
    "afternoon": "14:00",
    "night": "20:00"
}

def dispense_pill():
    print("Dispensing pill...")
    servo.start(2.5)
    time.sleep(1)
    servo.ChangeDutyCycle(12.5)
    time.sleep(1)
    servo.ChangeDutyCycle(2.5)
    servo.stop()
    print("Pill dispensed successfully!")

def send_sms(message):
    message = twilio_client.messages.create(
        body=message,
        from_=TWILIO_PHONE,
        to=USER_PHONE
    )
    print("SMS Alert Sent: ", message.sid)

def check_medication_time():
    now = datetime.now().strftime("%H:%M")
    for time_slot, time_value in med_schedule.items():
        if now == time_value:
            print(f"Time for {time_slot} medication.")
            dispense_pill()
            send_sms(f"Time to take your {time_slot} medication.")

def main():
    try:
        while True:
            check_medication_time()
            time.sleep(60)  # Check every minute
    except KeyboardInterrupt:
        print("Stopping DoseEase system.")
        GPIO.cleanup()

if __name__ == "__main__":
    main()
