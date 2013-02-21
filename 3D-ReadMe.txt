Using the DIWire Bender to print a 3D shape based on commands:

1.) The arduino program must me uploaded to the arduino board

2.) The computer running the 3D Processing program has the necessary libraries
    and is plugged into the arduino with the appropriate serial port defined

3.) The processing program calls upon a text file of commands named wiretest.txt
    within the same folder. the text file of commands is formated as follows

feed length (tab) xy bend angle (tab) z bend angle (return)

for example:
37	-90	0
37	90	30
20	90	0
37	-90	-20
20	-90	0
37	90	0
20	90	100
37	-90	0
37	90	0



note that no single feed or bend angle can exceed 124
