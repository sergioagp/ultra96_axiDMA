# Using the AXI DMA in Vivado
In this design, we’ll use the DMA to transfer data from memory to an IP block and back to the memory. In principle, the IP block could be any kind of data producer/consumer such as an ADC/DAC FMC, but in this tutorial we will use a simple FIFO to create a loopback. After, you’ll be able to break the loop and insert whatever custom IP you like.
![](https://i0.wp.com/www.fpgadeveloper.com/wp-content/uploads/2014/08/fpga_developer_20140806_130447.png)

The block diagram above illustrates the design that we’ll create. The processor and DDR memory controller are contained within the Zynq PS. The AXI DMA and AXI Data FIFO are implemented in the Zynq PL. The AXI-lite bus allows the processor to communicate with the AXI DMA to setup, initiate and monitor data transfers. The AXI_MM2S and AXI_S2MM are memory-mapped AXI4 buses and provide the DMA access to the DDR memory. The AXIS_MM2S and AXIS_S2MM are AXI4-streaming buses, which source and sink a continuous stream of data, without addresses.

Notes:

-   MM2S stands for Memory-Mapped to Streaming, whereas S2MM stands for Streaming to Memory-Mapped.
-   When Scatter-Gather is used, there is an extra AXI bus between the DMA and the memory controller. It was left out of the diagram for simplicity.

## Requirements
Before following this tutorial, you will need to do the following:
-   [Vivado](http://www.xilinx.com/products/design-tools/vivado/)  Instructions were written for version 2018.3, but the source code will be maintained to the latest version
-   [Ultra96v2](http://zedboard.org/product/ultra96-v2-development-board)

