# Using the AXI DMA in Vivado
In this design, we’ll **use the DMA to transfer data from memory to an IP block and back to the memory**. In principle, the IP block could be any kind of data producer/consumer such as an ADC/DAC FMC, but in this tutorial we will use a **simple FIFO to create a loopback**. After, you’ll be able to break the loop and insert whatever custom IP you like.
![](https://i0.wp.com/www.fpgadeveloper.com/wp-content/uploads/2014/08/fpga_developer_20140806_130447.png)

The block diagram above illustrates the design that we’ll create. The processor and DDR memory controller are contained within the Zynq PS. The AXI DMA and AXI Data FIFO are implemented in the Zynq PL. The AXI-lite bus allows the processor to communicate with the AXI DMA to setup, initiate and monitor data transfers. The AXI_MM2S and AXI_S2MM are memory-mapped AXI4 buses and provide the DMA access to the DDR memory. The AXIS_MM2S and AXIS_S2MM are AXI4-streaming buses, which source and sink a continuous stream of data, without addresses.

Notes:

-   MM2S stands for Memory-Mapped to Streaming, whereas S2MM stands for Streaming to Memory-Mapped.
-   When Scatter-Gather is used, there is an extra AXI bus between the DMA and the memory controller. It was left out of the diagram for simplicity.

## Requirements
Before following this tutorial, you will need to do the following:
-   [Vivado](http://www.xilinx.com/products/design-tools/vivado/)  Instructions were written for version 2018.3, but the source code will be maintained to the latest version
-   [Ultra96v2](http://zedboard.org/product/ultra96-v2-development-board)

## Step-by-step
### Start from the base project
1. Start by creating a base system project for the Ultra96v2

### Add the AXI DMA
1.  Open the base project in Vivado.
2.  In the Flow Navigator, click ‘Open Block Design’.
3. The block diagram should open and you should only have the Zynq PS in the design.
4.  Click the ‘Add IP’ icon and double click '**AXI Direct Memory Access**' from the catalog.
### Connect the Memory-mapped AXI buses
1.  The DMA block should appear and designer assistance should be available. Click the ‘Run Connection Automation’ link .
2. Click ‘OK’ in the window that appears. Vivado will connect the AXI-lite bus,  of the DMA to the General Purpose AXI Interconnect of the PS.
3. Your block diagram should now look like this :
4. Our PS doesn’t seem to have a high-performance AXI slave interface, so we need to change the Zynq configuration to enable one. Double click on the Zynq block.
5. Select ‘PS-PL Configuration’, open the ‘HP Slave AXI Interface’ branch and tick the ‘S AXI HP0 interface’ to enable it. Then click OK.
6. The high-performance AXI slave ports should now be visible in the block diagram, and designer assistance should be available. Click the ‘Run Connection Automation’ link and select ‘/processing_system7_0/S_AXI_HP0’ from the drop-down menu.
7. Now  connect the AXI buses M_AXI_SG, M_AXI_MM2S and M_AXI_S2MM of the DMA to S AXI HP0, the high performance AXI slave interface on the PS. 
Now all the memory-mapped AXI buses are connected to the DMA. Now we only have to connect the AXI streaming buses to our loopback FIFO and connect the DMA interrupts.
### Add the FIFO

1.  Click the ‘Add IP’ icon and double click ‘AXI4-Stream Data FIFO’ from the catalog.
2. The FIFO should be visible in the block diagram. Now we must connect the AXI-streaming buses to those of the DMA. Click the ‘S_AXIS’ port on the FIFO and connect it to the ‘M_AXIS_MM2S’ port of the DMA.
3. Then connect the ‘M_AXIS’ port on the FIFO and connect it to the ‘S_AXIS_S2MM’ port of the DMA.
4. Now we must connect the FIFO clock and reset. 

### Remove the AXI-Streaming status and control ports of the DMA

In our design, we won’t need the AXI-Streaming status and control ports which are used to transmit extra information alongside the data stream. You might use them if you were connecting to the AXI Ethernet core or a custom IP that made use of them.

1.  In the block diagram, double click the AXI DMA block.
2.  Un-tick the ‘Enable Control / Status Stream’ option and click OK.

### Connect the DMA interrupts to the PS

Our software application will test the DMA in polling mode, but to be able to use it in interrupt mode, we need to connect the interrupts ‘mm2s_introut’ and ‘s2mm_introut’ to the Zynq PS.

1.  First we have to enable interrupts from the PL. Double click the Zynq block and select the Interrupts tab.
2.  Tick ‘Fabric Interrupts’ and ‘IRQ_F2P[15 :0]’ to enable them, and click OK.
3.  Click the ‘Add IP’ icon and double-click ‘Concat’ from the catalog.
4. Connect the ‘dout’ port of the Concat to the ‘IRQ_F2P’ port of the Zynq PS.
5. Connect the ‘mm2s_introut’ port of the DMA to the ‘In0’ port of the Concat.
6. Connect the ‘s2mm_introut’ port of the DMA to the ‘In1’ port of the Concat.
### Include the OCM segment in the memory mapping
On Ultra96 we may optionally _include the OCM (on-chip_ _memory) segment_ in the memory mapping (excluded by default and causing non-critical warnings):
1. Access the Adress Editor
2. Under the folder 'Exluded Adress Segments' DATA_SG, right-click on zynq_ultra_ps_e_0 and click on 'Include Segment'

![](https://www.element14.com/community/servlet/JiveServlet/downloadImage/293607990-2894-658475/vivado-ZU-include-ocm-segment.png)

### Validate and build the design

1.  From the menu select Tools->Validate Design.
2. You should get this message saying that validation was successful.
3. Our block diagram now looks like this :
![](https://www.element14.com/community/servlet/JiveServlet/downloadImage/293607990-2894-658468/vivado-ZU-axi-data-fifo.png)
4. In the Flow Navigator, click ‘Generate Bitstream’.

### Export the hardware design to SDK
Once the bitstream has been generated, we can export our design to SDK where we can develop the software application that will setup a DMA transfer, wait for completion and then verify the loopback.

### Create a Software application
To make things easy for us, we’ll use the template for the hello world application and then modify it to test the AXI DMA.

### Modify the Software Application

We need to modify the hello world software application to test our DMA.

1.  From the Project Explorer, open the  `hello_world/src`  folder. Open the “helloworld.c” source file.
2.  Replace all the code in this file with the code that you will find here:  `Xilinx/SDK/(version)/data/embeddedsw/XilinxProcessorIPLib/drivers/axidma_v(ver)/examples/xaxidma_example_sg_poll.c`
3.  Save and close the file. The application should build automatically.

> The application source code we are using in this tutorial is one of the many valuable examples provided by Xilinx in the installation files. If you didn’t know about those examples, I suggest you check it out every time you start playing with a new IP core.

### Enable UART
1. In the Board Support Package Settings, select standalone.
2. Change the stdin and stdout value to `psu_uart_1`. This is done to align with the Ultra96’s UART Serial connection. Select OK.

### Boot from microSD using FSBL
1. Create the FSBL
2. Test the design on the hardware
3. Prepare the Boot Image
4. Write and boot from microSD

The application will be loaded on the Zynq PS and it will be executed. Look out for the results in your terminal window!
![fpga_developer_20140806_122245](https://i0.wp.com/www.fpgadeveloper.com/wp-content/uploads/2014/08/fpga_developer_20140806_122245.png?resize=843%2C249)

That's All Folks!!

## Resources
- [Test Procedure and Example Project: AXI DMA](https://www.element14.com/community/roadTestReviews/2894/l/avnet-ultra96-dev-board-review)
- [Using the AXI DMA in Vivado](http://www.fpgadeveloper.com/2014/08/using-the-axi-dma-in-vivado.html)
