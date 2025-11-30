## ESP32-FPGA-Cam-System
Real-time image processing system using ESP32-CAM and FPGA. Features live video capture, SDRAM frame buffering, VGA display output, and accelerometer-based image filtering with edge detection capabilities. Implements multi-processor architecture for parallel image manipulation




## Task Summary
# 1. VGA Display System

Implement a 25 MHz VGA pixel clock using a PLL.

Build a 320×240, 4-bit-per-pixel dual-port RAM pixel buffer.

Create a VGA controller that reads pixels and generates valid sync signals.

Display blank screens and test patterns to validate the graphics pipeline.

# 2. Nios II Processing System

A full Nios II-based subsystem is built using Platform Designer:

Includes:

Nios II/f processor

On-chip RAM (100 KB)

JTAG UART for debugging

PIOs for switches, LEDs, keys, HEX displays

PIO interface to the pixel buffer (address + pixel data)

Hardware microsecond counter for benchmarking

SDRAM controller to store entire camera frames

The processor can generate images in software and write them to the pixel buffer for display.

# 3. ESP32-CAM (ESP-CAM) Integration

Extend the SPI controller to communicate with the ESP32-CAM.

Configure the camera via SPI startup commands.

Receive 320×240 grayscale images over SPI.

Store raw camera frames in SDRAM before transferring to the pixel buffer.

# 4. Benchmarking

Use the microsecond timer to measure:

Frame capture time

Frame transfer rate

End-to-end frame display time

Compute and show calculated FPS on HEX displays.

# 5. Accelerometer Integration

Modify the SPI controller to communicate with both the ESP32-CAM and the onboard accelerometer.

Read live X/Y/Z acceleration measurements.

Configure double-tap interrupts.

Use tap events and acceleration data for user controls in later tasks.

# 6. Image Downscaling & Quad Display Mode

To support displaying 4 image versions at once:

Reconfigure ESP32-CAM to stream 160×120 resolution via SPI.

Store multiple processed frames sequentially in SDRAM.

Implement a 2×2 quad display, where four frames are shown simultaneously:

Default feed

Flipped feed

Blurred feed

Edge-detected feed

At this stage, identical placeholder frames may be used.

# 7. User Interface & Display Mode Switching

Two display modes must be implemented:

Quad Image Mode

User selects which frame appears in each quadrant.

require using accelerometer rotation to rotate/shift quadrant images.

Switches or accelerometer can control selection.

Single Image Mode

A single chosen image is displayed full-screen (320×240).

Switching between images must support:

Switch-based selection 

Double-tap accelerometer selection 

General Requirements

KEYs must toggle between quad mode and single mode.

SW[9:8] must be reserved.

# 8. Image Processing Algorithms

All processing occurs on the Nios II processor.

# 8A. Image Flipping

Flip input image across both X and Y axes.

Store result in a dedicated buffer.

# 8B. Convolution-based Filters

Implement a general-purpose 3×3 convolution function, then use it to produce:

Gaussian Blur

Kernel with all values = 1/9

Produces a smoothed image.

Sobel Edge Detection

Compute Gx using Sobel X kernel.

Compute Gy using Sobel Y kernel.

Combine using G = |Gx| + |Gy|.

Threshold output to produce binary edge mask.

# 8C. Processing Efficiency

Only run the image-processing algorithms for frames being displayed.

Use microsecond counter for benchmarking:

Default frame time

Flipped frame time

Blurred frame time

Edge-detected frame time

Quad-mode frame times (5 variations)

# 9. Communication Parallelism

Add a second Nios II processor dedicated solely to communication tasks.

Requirements

Add a second Nios processor to handle all SPI communication:

ESP-CAM frame transfer

Accelerometer data transfer

Main (original) processor:

Handles all image processing and display

Both processors must:

Use SDRAM for code and data (no on-chip memory)

Transfer data safely and efficiently without corruption

Run concurrently (true parallel execution)

You must implement a safe inter-processor communication method:

Mutexes

Mailboxes / message passing

Shared memory with locks

System must achieve real performance improvement.

# 10. Optimisations

# 10A. Display Parallelism

Add a third Nios II processor.

This processor must exclusively handle data movement from SDRAM to the pixel buffer (VGA).

Must explain how the display processor works and how it impacts performance.

# 10B. RGB Video

Modify your system to use the ESP-CAM in 12-bit RGB mode:

Data format: RRRR-GGGG-BBBB

Update ESP-CAM settings

Update VGA controller (minor modification)

Update image processing:

Flip: handle 12-bit pixels

Blur: apply convolution three times (R/G/B separately)

Edge detect:

Convert RGB → greyscale using
Gray = 0.299R + 0.587G + 0.114B

Then apply Sobel

Analyse performance impact.

# 10C. Packed Video Data

Configure ESP-CAM to send packed pixel data.

(Format described in background document.)

Modify SPI and frame unpacking logic.

Demonstrate understanding of how packing changes performance and memory bandwidth.

# 10D. Image Processing Parallelism

Add at least one more processor (in addition to Task 1) dedicated to:

Splitting image-processing tasks across processors OR

Parallelising convolution (e.g., different regions or rows)

You must:

Show that the new processor is used

Show that the system processes images faster

# 10E. Convolution Optimisations

Optimise your Sobel edge-detection implementation:

Reduce redundant operations

Improve loop structure

Improve memory access patterns

Apply known C optimisation techniques

Provide benchmarking showing improvement

# 10F. Custom Instructions

Create and integrate a hardware multiplier custom instruction:

Ideally a combinational custom instruction with:

2 × 32-bit inputs

1 × 32-bit output

Use the custom instruction to speed up convolution multiplications.

Focus on utilising all 32 bits efficiently.


