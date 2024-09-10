# ESP32 I2C OLED Display Example

This project demonstrates how to interface an ESP32 microcontroller with an OLED display using the I2C protocol. The project uses the [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") library for display handling and the FreeRTOS framework for task management.

## Prerequisites

- ESP32 development board
- OLED display with I2C interface
- ESP-IDF development environment
- [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") library cloned in the [`components`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fcomponents%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "c:\Users\sular\OneDrive\Documents\Arduino\ESP32-U8G2Example\components") folder from [olikraus/u8g2](https://github.com/olikraus/u8g2)

## Project Structure

```
project-root/
├── components/
│   └── u8g2/  # Cloned u8g2 library
├── main/
│   ├── CMakeLists.txt
│   └── main.cpp
├── CMakeLists.txt
└── sdkconfig
```

## main.cpp Overview

### Includes

The file includes necessary headers for GPIO, I2C, FreeRTOS, and the [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") library.

```cpp
#include <stdio.h>
#include "driver/gpio.h"
#include "driver/i2c_master.h"
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "u8g2.h"
#include "esp_log.h"
```

### Definitions

Defines GPIO pins and I2C parameters.

```cpp
#define I2C_MASTER_SCL_IO           GPIO_NUM_4
#define I2C_MASTER_SDA_IO           GPIO_NUM_5
#define I2C_MASTER_NUM              I2C_NUM_0
#define I2C_MASTER_FREQ_HZ          400000
#define I2C_MASTER_TX_BUF_DISABLE   0
#define I2C_MASTER_RX_BUF_DISABLE   0
```

### Global Variables

Declares global variables for I2C bus and device handles, and the [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") display structure.

```cpp
static const char *TAG = "U8G2";
i2c_master_bus_handle_t i2c_bus = NULL;
i2c_master_dev_handle_t i2c_dev = NULL;
u8g2_t u8g2;
```

### I2C Initialization

Initializes the I2C master bus and device.

```cpp
esp_err_t i2c_master_init(void)
{
    // Configuration and initialization code
}
```

### GPIO and Delay Functions

Implements GPIO and delay functions required by the [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") library.

```cpp
uint8_t u8x8_gpio_and_delay_esp32(u8x8_t *u8x8, uint8_t msg, uint8_t arg_int, void *arg_ptr)
{
    // Implementation code
}
```

### I2C Byte Communication

Handles I2C byte communication for the [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") library.

```cpp
uint8_t u8x8_byte_esp32_i2c(u8x8_t *u8x8, uint8_t msg, uint8_t arg_int, void *arg_ptr)
{
    // Implementation code
}
```

### Display Initialization

Initializes the OLED display and draws initial content.

```cpp
void u8g2_display_init(u8g2_t *pu8g2)
{
    // Initialization and drawing code
}
```

### Main Application

The main application initializes the I2C bus and the display, then enters an infinite loop to update the display.

```cpp
extern "C" void app_main(void)
{
    // Main application code
}
```

## How to Build and Run

1. Clone the [`u8g2`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fmain%2Fmain.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A22%2C%22character%22%3A7%7D%7D%5D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "Go to definition") library into the [`components`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2Fcomponents%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "c:\Users\sular\OneDrive\Documents\Arduino\ESP32-U8G2Example\components") folder:
   ```sh
   git clone https://github.com/olikraus/u8g2 components/u8g2
   ```

2. Build the project:
   ```sh
   idf.py build
   ```

3. Flash the project to the ESP32:
   ```sh
   idf.py flash
   ```

4. Monitor the output:
   ```sh
   idf.py monitor
   ```

## License

This project is licensed under the MIT License. See the [`LICENSE`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fsular%2FOneDrive%2FDocuments%2FArduino%2FESP32-U8G2Example%2FLICENSE%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%229f974e42-294e-4a49-bd0a-89330de8ddbd%22%5D "c:\Users\sular\OneDrive\Documents\Arduino\ESP32-U8G2Example\LICENSE") file for details.

## Acknowledgments

- [olikraus/u8g2](https://github.com/olikraus/u8g2) for the display library.
- Espressif Systems for the ESP-IDF framework.