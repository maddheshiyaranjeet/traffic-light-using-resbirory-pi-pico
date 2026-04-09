🚦 Smart Traffic Light  (Raspberry Pi Pico)

This project transforms a Raspberry Pi Pico into an intelligent traffic management system. It simulates a standard intersection cycle using **Red, Yellow, and Green LEDs**, while adding real-world complexity with a **pedestrian crossing button**. 

## 📝 Project Description
Unlike a basic timer-based light, this setup utilizes **Hardware Interrupts (IRQ)**. This allows the Pico to "listen" for a button press on GP16 even while the main code is running. To ensure the system remains responsive, the code uses a **non-blocking sleep method**—checking the "pedestrian flag" every 100ms rather than pausing entirely for five seconds.

The system mimics real-world safety by completing a shortened "Green" phase before transitioning safely to "Yellow" and "Red" once a crossing is requested.

## ✨ Features
*   **Standard Cycle:** Red (7s) → Green (5s) → Yellow (2s).
*   **Interrupt-Driven Button:** Uses hardware interrupts to detect pedestrian requests instantly.
*   **Safety Logic:** Ensures the light doesn't skip directly to Red, preventing "digital accidents."
*   **Non-Blocking Delay:** High responsiveness even during sleep cycles.

---

## 🛠 Hardware Requirements
| Component | Quantity | Connection (Pico Pin) |
| :--- | :--- | :--- |
| **Raspberry Pi Pico** | 1 | Microcontroller |
| **Red LED** | 1 | GP15 |
| **Yellow LED** | 1 | GP14 |
| **Green LED** | 1 | GP13 |
| **Push Button** | 1 | GP16 |
| **330Ω Resistors** | 3 | For LEDs |
| **Breadboard & Wires**| 1 set | For prototyping |

---

## 🔌 Circuit Wiring
*   **LEDs:** Connect the Anode (+) of each LED to its GPIO pin and the Cathode (-) to **GND** via a 330Ω resistor.
*   **Button:** Connect one side to **3.3V (Pin 36)** and the other side to **GP16**. The code uses an internal `PULL_DOWN` resistor, so no external resistor is needed for the button.

---

## 💻 Source Code (MicroPython)
Save this code as `main.py` on your Raspberry Pi Pico.

```python
import machine
import utime

# Pin Setup
red = machine.Pin(15, machine.Pin.OUT)
yellow = machine.Pin(14, machine.Pin.OUT)
green = machine.Pin(13, machine.Pin.OUT)
button = machine.Pin(16, machine.Pin.IN, machine.Pin.PULL_DOWN)

pedestrian_waiting = False

# Interrupt Service Routine
def button_handler(pin):
    global pedestrian_waiting
    pedestrian_waiting = True

button.irq(trigger=machine.Pin.IRQ_RISING, handler=button_handler)

def run_system():
    global pedestrian_waiting
    while True:
        # RED PHASE
        red.value(1); yellow.value(0); green.value(0)
        utime.sleep(7) 
        pedestrian_waiting = False 
        
        # GREEN PHASE
        red.value(0); yellow.value(0); green.value(1)
        for _ in range(50): # 5 second total wait, checked every 100ms
            utime.sleep(0.1)
            if pedestrian_waiting:
                break 

        # YELLOW PHASE
        red.value(0); yellow.value(1); green.value(0)
        utime.sleep(2)

try:
    run_system()
except KeyboardInterrupt:
    red.value(0); yellow.value(0); green.value(0)

🚀 Expansion Ideas
Audio Signal: Add a Piezo buzzer on GP17 to beep when the Red light is active for pedestrians.

LCD Display: Add an I2C LCD to show a "Seconds Remaining" countdown.

Dual-Lane: Sync two sets of LEDs to simulate a 4-way intersection.




🚀 Future Scope & Advanced Enhancements
1. Adaptive Traffic Control (Sensor Integration)
In real-world scenarios, lights stay green longer if there are more cars. You can simulate this by adding:

Ultrasonic Sensors (HC-SR04): Positioned at the "stop line" to detect if a car is actually waiting.

Induction Loop Simulation: Use a magnetometer to detect metal (like a small toy car) passing over a specific spot.

2. IoT & Remote Monitoring (Pico W)
If you upgrade to the Raspberry Pi Pico W, you can add Wi-Fi capabilities to create a "Smart City" node:

Web Dashboard: Host a simple web page on the Pico to see real-time status or manually override the lights from your phone.

Data Logging: Send "pedestrian request" counts to a cloud service (like Adafruit IO) to analyze peak traffic times.

3. Emergency Vehicle Priority (Preemption)
Simulate a "Priority Mode" for ambulances or fire trucks.

Sound Detection: Use a microphone module to detect the frequency of a siren.

RF/IR Remote: Use an Infrared receiver so a "driver" (you with a remote) can force the light to turn Green immediately for emergency passage.

4. Smart City Grid (Mesh Networking)
Real traffic lights don't work in isolation; they are synchronized.

Pico-to-Pico Communication: Use two Picos connected via UART or I2C. When Light A turns Red, it sends a signal to Light B to turn Green, simulating a 4-way intersection without crashing.


Getty Images
5. Computer Vision Integration
Connect the Pico to a computer running OpenCV.

Object Detection: The computer identifies "Car" or "Person" via a webcam and sends a command to the Pico via Serial (USB) to change the lights based on actual visual density.

6. Energy Efficiency & Sustainability
Solar Power: Integrate a small 5V solar panel and a LiPo battery charger to make the unit completely off-grid.

Dimming Logic: Use a Photoresistor (LDR) to detect ambient light levels. The Pico can then use PWM (Pulse Width Modulation) to dim the LEDs at night to save powe
