Adafruit Slider Trinkey ProjectA brief one-sentence description of what this project does.DemoA short video or GIF demonstrating the project in action.Note: Replace the link above with your own YouTube video ID.📝 DescriptionProvide a more detailed overview of your project here. Explain what problem it solves, what its main features are, and how it works.🚀 FeaturesFeature 1: Capacitive touch slider input.Feature 2: Two RGB NeoPixel LEDs for visual feedback.Feature 3: USB C connectivity.Add any other features specific to your project.💻 Code SnippetsBelow are some basic examples to get you started. You can replace these with your own code.CircuitPythonThis example shows how to read the slider's value and change the NeoPixel colors.import time
import board
import touchio
from adafruit_trinkey import Trinkey
import neopixel

# --- Setup ---
trinkey = Trinkey()
pixels = neopixel.NeoPixel(board.NEOPIXEL, 2)
touch = touchio.TouchIn(board.TOUCH)

# --- Main Loop ---
while True:
    slider_position = touch.raw_value

    print(f"Slider Value: {slider_position}")

    # Example: Change pixel color based on slider position
    if slider_position > 3000:
        pixels.fill((255, 0, 0)) # Red
    elif slider_position > 1000:
        pixels.fill((0, 255, 0)) # Green
    else:
        pixels.fill((0, 0, 255)) # Blue

    pixels.show()
    time.sleep(0.1)
Arduino (C++)This is a basic structure for an Arduino sketch for the Slider Trinkey.#include <Adafruit_NeoPixel.h>
#include <Adafruit_FreeTouch.h>

#define PIN_NEOPIXEL 1
#define PIN_TOUCH    A0 // Or the correct analog pin for the slider

// --- Setup ---
Adafruit_NeoPixel pixels(2, PIN_NEOPIXEL, NEO_GRB + NEO_KHZ800);
Adafruit_FreeTouch touch(PIN_TOUCH, OVERSAMPLE_4, RESISTOR_50K, FREQ_MODE_PROBE);

void setup() {
  Serial.begin(115200);
  pixels.begin();

  if (!touch.begin()) {
    Serial.println("Failed to start FreeTouch!");
  }
}

// --- Main Loop ---
void loop() {
  int slider_value = touch.measure();

  Serial.print("Slider Value: ");
  Serial.println(slider_value);

  // Example: Set brightness based on slider value
  if (slider_value > 500) {
    pixels.setBrightness(255);
  } else {
    pixels.setBrightness(50);
  }
  pixels.fill(pixels.Color(0, 150, 255));
  pixels.show();

  delay(100);
}
🛠️ InstallationHardware:Adafruit Slider TrinkeyUSB-C CableSoftware:CircuitPython for Slider TrinkeyMu Editor or any other text editor.Add your own detailed setup instructions here.📜 LicenseThis project is licensed under the MIT License.
