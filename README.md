#include <IRremoteESP8266.h>
#include <IRsend.h>


const uint16_t ir_led_pin = 4;  // IR LED connected to GPIO 4
IRsend irsend(ir_led_pin);


// Timer variables
hw_timer_t * timer = NULL;
volatile bool sendSignal = false;


void IRAM_ATTR onTimer() {
  sendSignal = true;  // Set flag when timer is triggered
}


void setup() {
  Serial.begin(115200);
  irsend.begin();
  Serial.println("IR Transmitter Ready!");


  // Set up timer to trigger every 1000ms (1 second)
  timer = timerBegin(0, 80, true);  // Timer 0, divide by 80 to get 1ms interval
  timerAttachInterrupt(timer, &onTimer, true);  // Trigger onTimer() on timer interrupt
  timerAlarmWrite(timer, 1000000, true);  // Trigger the timer every 1000000 microseconds (1 second)
  timerAlarmEnable(timer);  // Enable the timer
}


void loop() {
  if (sendSignal) {
    sendSignal = false;  // Reset the flag
    Serial.println("Sending IR Signal...");
    
    // Send "ON" signal
    irsend.sendNEC(0xFFA25D, 32);  // Example signal for "ON"
    Serial.println("Sent: 0xFFA25D");
    
    // Send "OFF" signal after a small delay
    delay(500);  // Small delay before sending the "OFF" signal
    irsend.sendNEC(0xFF629D, 32);  // Example signal for "OFF"
    Serial.println("Sent: 0xFF629D");
  }
}
