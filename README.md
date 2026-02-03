# ELEC2645 - Joystick Input with LCD Display

This project demonstrates embedded systems programming on the STM32L476 Nucleo board, focusing on:
- **Analog Input Processing** - Reading joystick input from ADC channels
- **Coordinate Transformations** - Converting between Cartesian and polar coordinates
- **Circle Mapping** - Optional square-to-circle mapping for uniform control feel
- **LCD Screen Control** - Visualizing joystick data on an ST7789V2 SPI display

The most important file is [Core/Src/main.c](Core/Src/main.c) which contains the main application logic.

## The Project

The program reads a 2-axis analog joystick, processes the input into useful coordinate representations, and displays the results on the LCD. Typical output includes:
1. **Raw ADC values** - The 12-bit ADC readings from the joystick X and Y axes
2. **Angle and Magnitude** - Compass heading (0-360°) and magnitude (0.0-1.0)
3. **Direction** - Discrete 8-direction output (N, NE, E, SE, S, SW, W, NW, or CENTRE)
4. **Visual Representation** - A dot that moves within a circular frame

## Setup Instructions

### Prerequisites

1. **Completed previous labs**
	- Blinky and LCD Test to verify hardware
	- Familiarity with ADC and GPIO basics

2. **Configure the Project**
	- Open the JoystickNewProject2 folder in VS Code
	- When prompted to configure the CMake project, choose **Yes**
	- Select **Debug** configuration when prompted

3. **Verify Hardware Connection**
	- Connect the Nucleo board via USB
	- Ensure the board appears under the STM32 extension in the Run and Debug sidebar

4. **Build and Run**
	- Build from the status bar or Run and Debug panel
	- Use **STM32Cube: STLink GDB Server** to flash and debug

### Troubleshooting

- **Board not detected**: Check ST-Link drivers and USB cable
- **Build errors**: Verify joystick and LCD include paths are present in CMake
- **Joystick not responding**: Confirm ADC channels are configured for X (ADC_CHANNEL_1) and Y (ADC_CHANNEL_2)
- **Joystick not centred correctly**: Power the joystick from 3.3V, not 5V
- **LCD display issues**: Confirm SPI2 and GPIO pin assignments match the wiring table

## Hardware Configuration

### Analog Joystick Connection

| Joystick Pin | Signal | Nucleo Pin | Purpose |
|-------------|--------|-----------|---------|
| VCC         | Power (3.3V) | VDD | Joystick power supply |
| GND         | Ground | GND | Ground reference |
| X-axis      | Analog X | PA1 (ADC1 CH1) | Horizontal position |
| Y-axis      | Analog Y | PA2 (ADC1 CH2) | Vertical position |
| Button      | (Optional) | - | Can be added for digital input |

### LCD Display Connection (ST7789V2)

| LCD Pin | Signal | Nucleo Pin | Purpose |
|---------|--------|-----------|---------|
| VDD     | Power (3.3V-5V) | VDD | Display power supply |
| GND     | Ground | GND | Ground reference |
| MOSI    | Serial Data | PB15 | SPI Master Output Slave Input |
| SCK     | Clock | PB13 | SPI Clock signal |
| CS      | Chip Select | PB12 | SPI Chip Select |
| DC      | Data/Command | PB11 | Command vs Data mode |
| BL      | Backlight | PB1 | Backlight control (active high) |
| RST     | Reset | PB2 | Display reset (active low) |

### Serial Communication
- **UART Interface**: USB connection via ST-Link debugger
- **Baud Rate**: 115200
- **Purpose**: Debug output via `printf()` and viewing calibration information

## Software Architecture

### Key Source Files

| File | Purpose |
|------|---------|
| [Core/Src/main.c](Core/Src/main.c) | **Main application** - Joystick initialization, calibration, display loop |
| [joystick/joystick.h](joystick/joystick.h) | Joystick driver header - API documentation and type definitions |
| [joystick/joystick.c](joystick/joystick.c) | Joystick driver implementation - ADC reading, circle mapping, coordinate transformations |
| [Core/Src/adc.c](Core/Src/adc.c) | ADC peripheral initialization |
| [Core/Src/gpio.c](Core/Src/gpio.c) | GPIO peripheral initialization |

### Joystick Library

The joystick driver provides a complete abstraction for reading analog joystick input:

**Key Data Structures:**

- **Joystick_t** - Contains all joystick state:
  - `x_raw`, `y_raw` - Raw 12-bit ADC values
  - `coord` - Cartesian coordinates (-1.0 to 1.0) before circle mapping
  - `coord_mapped` - Cartesian coordinates after circle mapping (uniform control feel)
  - `angle` - Compass heading 0-360° (0°=North, 90°=East, etc.)
  - `magnitude` - Distance from center 0.0-1.0
  - `direction` - 8-direction enum (N, NE, E, SE, S, SW, W, NW, CENTRE)

**Key Functions:**

- `Joystick_Init()` - Initialize ADC channels and configuration
- `Joystick_Calibrate()` - Find center position (call while joystick is centered)
- `Joystick_Read()` - Read current joystick state (call in main loop)

See [joystick/joystick.h](joystick/joystick.h) for complete API documentation.

### LCD Driver

The project includes the ST7789V2 LCD driver for the 240×240 display:
- **Location**: [ST7789V2_Driver_STM32L4/](ST7789V2_Driver_STM32L4/)
- **Key Functions**:
  - `LCD_Draw_Circle(x, y, radius, color, fill)` - Draw a circle
  - `LCD_printString(x, y, text, size, color)` - Display text
  - `LCD_Fill_Buffer(color)` - Clear the display
  - `LCD_Refresh()` - Update the physical display
  - `LCD_Invert()` - Invert display colors
  - `LCD_Normal_Mode()` - Return to normal display mode

## Understanding the Code

### Initialization (Before Main Loop)

```c
// Create joystick configuration
Joystick_cfg_t joystick_cfg = { ... };

// Initialize ADC and find center position
Joystick_Init(&joystick_cfg);
Joystick_Calibrate(&joystick_cfg);
```

### Main Loop

```c
while (1) {
	 // Read all joystick data (raw ADC, processed, cartesian, polar, direction)
	 Joystick_Read(&joystick_cfg, &joystick_data);

	 // Use mapped coordinates for uniform control
	 float mapped_x = joystick_data.coord_mapped.x;
	 float mapped_y = joystick_data.coord_mapped.y;

	 // Update the display
	 LCD_Refresh(&cfg0);
}
```

### Available Data

When you need to check how far the joystick is deflected:
- `joystick_data.magnitude` - Value 0.0-1.0 indicating how far from center
  - 0.0 = joystick at center
  - 1.0 = joystick fully deflected

## Tips for Success

1. **Test Your Understanding**: Print `joystick_data.magnitude` to the serial monitor to see the values while moving the joystick
2. **Start Simple**: Validate ADC readings before adding display logic
3. **Debug**: Use breakpoints or `printf()` for quick checks
4. **Performance**: Limit full-screen LCD redraws when possible to keep refresh smooth

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Direction shows CENTRE when joystick is pushed" | Magnitude check missing or threshold too high | Verify magnitude threshold and direction logic |
| "Dot position is jerky or inverted" | Y-axis polarity issue | Check coordinate sign or display Y-axis direction |
| "Colours don't look right" | Wrong color value | Verify palette and LCD color constants |
| "Compilation errors about joystick.h" | Include path issue | Ensure joystick folder is in include path |
| "Display doesn't update" | LCD not initialised or refresh missing | Confirm LCD init and `LCD_Refresh()` in loop |

## Notes

Floating point printf support is enabled in the root [CMakeLists.txt](CMakeLists.txt). If RAM usage becomes critical, consider disabling it.
