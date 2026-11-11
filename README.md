<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->
![](../../workflows/gds/badge.svg) ![](../../workflows/docs/badge.svg) ![](../../workflows/test/badge.svg) ![](../../workflows/fpga/badge.svg)
![](./docs/bwsi.png)

# 6x6 CNN Accelerator 

The purpose of this project is to create an ASIC that computes convolutions for a 6x6 input image and a 3x3 kernel. This repository contains the GDS file required to tapeout this chip, as well as the source files and testbenches used to create it. We acknowledge the MIT BeaverWorks Summer Institute for their guidance and creation of this opportunity. 

Project Contributors: Tori Porat, Han Lu, Sharada Parameswaran, Michael Evans

## Architecture
The primary I/O are the loaded matrices and the computed matrix respectively. 

The loading procedure is as follows: First, the memory bank is selected using a binary selector, with writing `0` selecting the feature map, while writing `1` selects the kernel to be written. Then, `ld_addr` selects the address within the memory bank to write to. Finally, an 8 bit `ld_data` corresponding to the data in that address is loaded. Loading for both the kernel and the feature map completes the loading process. The reading process is simple, with a `rd_addr` to specify matrix element and a 32 bit integer output via `rd_data`.

The convolution computation comprises of 2 main parts: (1) flattening the matrix into column form for use in im2col and (2) using parallel MAC (multiply and accumulate) units to compute the dot products necessary for each shifted frame in the convolution. To achieve step 1, the memory bank was modified to manipulate the address from the transformed matrix into the corresponding address for the original feature map. This way, the MAC units believe that they are accessing an intermediate column matrix, while in reality they are still accessing the matrix that was loaded, thus saving space. Multiply and accumulate units are then instantiated, accessing these memory banks. For this 6x6 case, we elected to use 4 MAC units which balances space and efficiency. An FSM controls the current frame being computed, hence coordinating data flow. 

Optimizations: 
1. Rather than using combinational circuitry to calculate the addresses of the transformed matrix, we calculated them pre-computationally and hard-wired them on the circuit, since the addresses are constant sequentially. 
2. Memory bank multiporting to allow for parallel MAC computation
3. Data resizing to reduce space, eliminating unneeded allocation.

Our Final Power Distribution:
<img width="499" height="254" alt="Final CNN ASIC Power Distribution" src="https://github.com/user-attachments/assets/cd6e8073-8c29-4958-bbd6-1dd44e38d02f" />

## Testbench
We decided to create two testbenches in order to facilitate debugging our Verilog code. We started with our smaller Matcol Module and primarily focused on verifying the correct tap address. This way, we can ensure that the correct calculations are being performed. Our overall CNN testbench includes loading the test images and kernels, checking all 16 cases using MAC multiplication (for a total of 144 outputs), and comparing each output to the expected feature map. Despite some issues here and there, we were able to create a testbench where all cases passed, increasing our confidence that the Verilog to GDSII flow would work properly. 

<img width="526" height="384" alt="Passed Testbench" src="https://github.com/user-attachments/assets/4c20b0a9-614f-41b0-954d-9601dcc59b75" />

Fun Fact! Single-layer CNNs can perform edge detection using Sobel Filters. To fully test our code, we wrote another testbench where we fed in a 16x16 image of a bunny (of course), and received an 14x14 output of the edges. This can be seen below. 

<img width="403" height="311" alt="Bunny Image" src="https://github.com/user-attachments/assets/e4952c9b-d7ed-4a6c-b4ad-a5d4ac9e8442" />
<img width="628" height="281" alt="Bunny edges" src="https://github.com/user-attachments/assets/5cfb77aa-b3cc-4064-be3b-596516266c49" />

## Outputs
After running our Verilog through OpenLane and completing extra optimizations, we were able to eliminate all antenna errors, as well as pass DRC and LVS Checks. 

<img width="838" height="248" alt="Checks passed" src="https://github.com/user-attachments/assets/887d5b07-1ac8-4cbf-8ced-ea526835f07e" />

This meant that our ASIC was finally done, and below are some views of the GDSII file in KLayout and the Tiny Tapeout viewer (https://gds-viewer.tinytapeout.com/) respectively. 

<img width="691" height="638" alt="KLayout ASIC View" src="https://github.com/user-attachments/assets/120ffc62-c668-440c-bda9-f86f9fbd5b6b" />

<img width="308" height="310" alt="Tiny Tapeout View 1" src="https://github.com/user-attachments/assets/619fa427-e260-4de8-9a87-f62aee23cdbc" />
<img width="541" height="529" alt="Tiny Tapeout View 2" src="https://github.com/user-attachments/assets/a4030e4d-9797-4853-8903-0acfdedbd6ab" />

<?xml version="1.0" encoding="UTF-8"?>
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 396.12 60.07">
  <defs>
    <style>
      .cls-1 {
        fill: #005894;
      }

      .cls-2 {
        fill: #0b2338;
      }
    </style>
  </defs>
  <g id="Layer_1-2" data-name="Layer 1">
    <g>
      <polygon class="cls-1" points="52.18 0 37.61 37.54 22.89 0 11.25 0 0 60.06 12.01 60.06 19.14 22.45 33.03 60.06 42.05 60.06 55.94 22.45 63.07 60.06 75.44 60.06 63.82 0 52.18 0"/>
      <rect class="cls-1" x="78.51" y="0" width="11.25" height="60.06"/>
      <polygon class="cls-1" points="92.83 0 92.83 9.75 106.05 9.75 106.05 60.06 117.23 60.06 117.23 9.75 130.45 9.75 130.45 0 92.83 0"/>
      <g>
        <path class="cls-1" d="M144.77,60.06h-11.25V0h26.72c1.1,0,3,.08,5.41,1.06,2.34.96,5.01,2.39,7.51,4.95,2.49,2.6,3.74,6.35,3.74,11.26s-1.35,8.96-4.05,11.56c-2.7,2.65-5.43,4.33-8.17,5.02-2.75.76-4.78,1.14-6.09,1.14l19.98,25.07h-13.53l-18.31-24.78v-10.81l11.18-.07h.61c1.55,0,3.1-.57,4.65-1.72,1.7-1.29,2.55-3.1,2.55-5.41,0-2.1-.7-3.82-2.11-5.17-1.44-1.31-3.34-1.96-5.7-1.96h-13.14v49.93Z"/>
        <polygon class="cls-1" points="181.64 0 216.39 0 216.39 9.75 192.89 9.75 192.89 25.15 216.17 25.15 216.17 34.9 192.89 34.9 192.89 50.31 216.39 50.31 216.39 60.07 181.64 60.07 181.64 0"/>
      </g>
    </g>
  </g>
  <g>
    <rect class="cls-2" x="251.59" y=".03" width="1.51" height="60"/>
    <rect class="cls-2" x="251.6" y=".03" width="1.51" height="60"/>
  </g>
  <g>
    <path d="M299.42,10.61v33.7h17.46v6.45h-25.13V10.61h7.67Z"/>
    <path d="M342.4,32.68v14.12c0,.95.06,2.61.56,3.95h-6.73l-.44-2.5c-2.95,2.06-5.84,3.06-8.95,3.06-4.89,0-7.9-3.28-7.9-7.67,0-6.56,5.45-10.06,16.57-11.34v-2.45c0-2.22-1.78-3.45-4.28-3.45s-4.5,1.06-6.28,3.56l-4.45-3.39c1.89-3,6.12-5.28,10.51-5.28,8.23,0,11.4,3.34,11.4,11.4ZM329.17,46.19c1.78,0,4.34-1.33,6.34-3.06v-6.23c-7.95,1.28-9.67,3.34-9.67,5.89,0,2.17,1.39,3.39,3.34,3.39Z"/>
    <path d="M354.45,10.61v13.9h.11c1.33-1.67,4.45-3.22,7.01-3.22,6.95,0,11.56,6.45,11.56,14.29,0,9.51-4.28,15.74-11.56,15.74-3.12,0-5.39-1.28-7.84-3.67l-.78,3.11h-5.39V10.61h6.9ZM354.45,43.19c1.06.83,2.22,2.56,5.34,2.56,4.11,0,6.45-3.28,6.45-9.45s-2.34-9.45-6.45-9.45c-1.95,0-3.73.95-5.34,2.45v13.9Z"/>
    <path d="M395.29,25.68l-4.34,3.78c-1.89-2.06-3.61-3.06-5.39-3.06-2.61,0-3.89,1.06-3.89,2.84,0,1.61,1.56,2.67,5.17,3.84,7.06,2.33,9.29,5.12,9.29,9.62,0,5.5-3.95,8.62-10.62,8.62-4.62,0-8.34-2-10.95-5.84l4.62-3.45c2.06,2.83,4.28,4.17,6.73,4.17s4.11-1.33,4.11-3.28c0-1.67-1.67-2.72-5.34-4-6.95-2.33-9.12-5.12-9.12-9.23,0-5.17,4.34-8.4,9.56-8.4,3.95,0,7.67,1.5,10.18,4.39Z"/>
  </g>
</svg>
