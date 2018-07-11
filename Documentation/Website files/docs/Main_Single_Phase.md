﻿![](https://lh4.googleusercontent.com/bHyVtnIbOT5By3YylKS4iocdPNwsTl6PEPraTGbRTqsFOMYN05V3H6-J-92oEPa5ftEJiD8SLzAtItHCpbYO694d3_w5eA1GTayN3QG0cacxumdhYM72tYXb05TESGom6g0fNrDL)
The VI 00 Main (single phase) combines all the of the VIs to act as a PMU and output data. Note that everything on the Front Panel is output, and all inputs are found through the sub VIs. Make sure to note:
![](https://lh5.googleusercontent.com/oOTpLmHACTNPmnMfEOMgHaq4zBM1B68IaR1gEThbfEWvWizqdzv7lauLjInQ4Ota6Cs9mg0pCPtGSo3sLa3H0rnGVU3WMb_HMJYvEl-YdV_qhp2a9FWyFlPxMPwSgT1I1H-ZvaX4)

![](https://lh3.googleusercontent.com/Pp2TQcI6E3rSK026zDvoaOSr6kF4gduzrlUUnTcZwWZfrDYRFV4lt8rM0ps68W6PfNZzvdI9pu8SMsgK47-l2ibB2kbFOH4yiC__4lsSZL5pD2BMcx2hKx0yk-X5q3GYTcL89hwS)

   Above is the first While loop of 00 Main (single phase). The purpose of the loop is to continuously take an analog input and store it in a queue. The VI starts at 1 by creating a queue with a Dynamic Data Type. The VI 10 DAQ Config (PFI0) at 2, sets the number of samples and the rate to sample, then creates a task. For all intents and purposes, the task data line acts as our data acquisition line, or the data recorded for processing. Look at [http://zone.ni.com/reference/en-XX/help/370689M-01/daqmxtutorial/newconceptsinnidaqmx/](http://zone.ni.com/reference/en-XX/help/370689M-01/daqmxtutorial/newconceptsinnidaqmx/) for more information on what a task is. Following the data line task out, the task is sent to the DAQmx Read (Analog 1d Wfm NChan NSamp) VI at 3, which finds our voltage analog input. This data out is converted from an array to a dynamic data line and sent forward. Additionally, the number of samples to record and the timeout is sent. Timeout is how long the VI will wait for an available signal. If time surpasses the timeout value, the VI will send an error. All errors and tasks are sent to the DAQmx Clear task or Error out (DAQ Loop) outside of the loop. If there is an error, or the Stop button on the Front Panel is pressed, the loop stops and errors are outputted. At 4, the VI 34 GPS Time from Config finds and returns a time value to be used for synchronous analysis. The value is multiplied by ten, rounded, and divided by ten to ensure that the value sent to the merge signals is only to the tenths place. The merge signal function takes the time, the voltage and a boolean value if the GPS is on Lock. The merge signal is now enqueued. If the loop finishes, the queue is released.

![](https://lh6.googleusercontent.com/L9Z7ax1BbjER3YcB4F2mZLXOQQxvOSV1FDxZXYl9zNqjiQNg84pUlfp0YU0FeyMCJAmuM5L05OwU_abyJRioSoxot816nQApfdEnMwMcH4RZ50jvt6UWq3R6IP8HU0qn7IY_T7_7)

   This is the second part of 00 Main (single phase). It analyses the data sent from the first portion through a queue. It starts at 1, with finding if there are any error in the queues and sends the elements in the buffer to the Front Panel to help debug. The data and any errors are dequeued and sent to a case structure. If there are no errors, the case structure is satisfied, else the errors are sent out. Once the data is inside the case structure at 2, the data is sent through 23 Input Scaling, which modifies the data based on input values from the user. The data is then split and sent to to Express VIs. The voltage is sent as a waveform, to the Front Panel, to the Tone Measurements which finds the Frequency and phase, and to the Amplitude and Level Measurements which returns the amplitude of the voltage. At 3,  the phase value is sent through 20 Limit 180 Degrees to ensure that the phase angle is between 0 and 180 degrees. Both the phase and frequency are sent as waveform outputs to the Front Panel. Below 3, the time data is converted back to a double, and sent to the Front Panel. The GPS Lock value is sent forward and will be discussed next. Note that this loop waits 1 ms to help limit the flow of output.

**![](https://lh4.googleusercontent.com/yzqOjPXFqWaQgPtarjh2Sj4N-MUGygGYmdx-6alVEFhFCSM8wwV4_wacfaJiPQNvrIgawC6fNyLENX8BGgxVRMmbkm418oeo6GfmtqpOzofn-M_E0xa9VZc-NuRGPrw0ucHLkD9g)**
     This section of the VI is still in the case structure that ensures that there are no errors before analysing the data. This section starts at 1 where the Frequency, Amplitude, and Phase are all sent to the Front Panel as output double values. 2 takes the GPS Lock boolean value and begins by converting it to a 1D array of the string, either T or F. The T or F is then concatenated with V1 and xx to produce a string sent to format a readable string at 4. Before this, at 3, the VI 44 Config File Read Telecoms, which returns the ID, IP, and Port value. The IP and Port are sent to the Front Panel as output. The ID is sent to the Format into String at 4 along with the Frequency, Amplitude, Phase, Time and concatenated string value. These are converted to the string as shown in the picture above of one string with values of two strings four, three decimal place floating point values all separated by a comma. Note the * on the end. The string value is sent to the VI 50 NMEA Calc Checksum, which returns a checksum value.

**![](https://lh3.googleusercontent.com/LUI3NI38KU3Sb-Am5CJOPZoEmqNkXjd--Tx_Mj6nPIPh5F_OKDRCbC-uVzEVqJ7rVtyU-YM5xvccJvg7p40u35AXCwD-BQW7GBuBNWxhwPQMWI4QM35B3pSez310f64xyDgfGJCS)**
    This portion of the VI finishes the analysis portion of the loop. It starts at 1, where the GPS string and the Checksum string are concatenated. Once the string is concatenated, it is sent to the UDP (User Datagram Protocol) write, along with the port and IP as net addresses. More can be found out about what a UDP is at: https://cs.nyu.edu/bacon/phd-thesis/diss/node32.html The GPS code is sent to the UDP, as well as output to the Front Panel as “Transmitted Data.” At 3, the UDP is opened for writing, with any errors and the port data sent to the write. The number is the timeout indicator. After 40 seconds, the write will timeout. Also note the all stop, which stops data collection, but does not exit the VI. Once the entire case structure is complete, the UDP is closed.

**![](https://lh3.googleusercontent.com/WtdBxR8DpkEVz6LBe7rTW25D0CWD1efTHZj_SXzpHfjrlPmUmag5VfTt0_7euA3yq47SpLRT7HPr8tLjJ5Um_Dz8hAuLYXEJcww5iGFjH-d9KRmPKb9dP0SrmpXRfZZNL_BEFC3r)**
    This portion of the VI is separate from the other two while loops. Each loop iteration starts by getting the user scale values from 43 Config File Get Input Scales. The second portion of the flat sequence is 30 GPS Time Fetch, which finds the time data from the GPS. Once complete, the final part of the flat sequence waits for 10 minutes. Because the flat structure is in a while loop, this process is repeated while the VI is running. 
