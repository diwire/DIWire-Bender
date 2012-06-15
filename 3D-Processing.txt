/* DIWire Bender
 * 3D Wire Bender by Pensa - www.PensaNYC.com
 * Written by Marco Perry. Email DIWire@PensaNYC.com for questions.
 * Drives on 3 Stepper Motors to bender wire in 3D space
 *
 * This file is part of the DIWire project. 
 * This file bends in 3D using commands (foward 50 mm, righ 90 degrees) read from a text file.
 *
 *  DIWire is a free software & hardware device: you can redistribute it and/or modify
 *  it's software under the terms of the GNU General Public License as published by
 *  the Free Software Foundation, version 3 of the License.
 *   
 *  The hardware portion is licenced under the Creative Commons-Attributions-Share Alike License 3.0
 *  The CC BY SA licence can be seen here: http://creativecommons.org/licenses/by-sa/3.0/us/
    

 *  DIWIre is distributed in the hope that it will be useful,
 *  but WITHOUT ANY WARRANTY; without even the implied warranty of
 *  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 *  GNU General Public License and CC-BY-SA for more details.

 *  You should have received a copy of the GNU General Public License
 *  along with DIWire.  If not, see <http://www.gnu.org/licenses/>. 
 *  and http://creativecommons.org/licenses/by-sa/3.0/us/legalcode
 *  
 *  No portion of this header can be removed from the code. 
 *  Now enjoy and start making something!
 */
 
//import these libraries from libraries folder
import processing.opengl.*;
import processing.serial.*;
Serial arduino;

/*
 * DIYER Bender
 * 3D Wire Bender by Pensa - www.PensaNYC.com
 * Written by Marco Perry. Call (718) 855 - 5354 for questions or support.
 * Drives on 3 Stepper Motors to bender wire in 3D space
 *
 */

/* the following program uploads a text file that contains a feed length , bend agle, z bend angle matrix (see readme.txt).
 The program shows an interactive 3D preview of the shape and sends the commands to the arduino for printing*/


int loopLength =0;
PVector[] pnt =  new PVector [255];
boolean mp = false;
float rotX=0;
float rotY=0;
float s=1;

void setup() {

  size (400, 400, OPENGL); //creates window with opengl library capabilities
  background (255);
  println(Serial.list());
  arduino = new Serial(this, Serial.list() [0], 9600); //sets arduino usb port
  println ();
  println ("SET THE SYSTEM TO HOME POSITION");
  println ("CLICK AND DRAG MOUSE TO ROTATE PREVIEW");
  println ("PRESS 1 TO PRINT");
  println ();
  String[] lines = loadStrings ("wiretest.txt"); //loads text file from folder
  loopLength = lines.length;
  pushMatrix();
  for (int i = 0; i < lines.length; i++) { //turns text file into an array
    String[] pieces = split(lines [i], '\t');
    pnt[i] = new PVector ((int (pieces[0])), (int (pieces[1])), (int (pieces[2])));
    println (pnt[i]);
  }
  popMatrix();
}

void draw() {
  translate (width/2, height/2, 0); // Sets draw location and scale
  //rotateX (PI);
  background (255);
  scale (s);
  if (mp) { //when mouse is pressed mouse location rotates the 3D coordinate system
    rotX = map (mouseY, 0, height, -PI, PI);
    rotY = map (mouseX, 0, width, -PI, PI);
  }
  rotateX (rotX);
  rotateY (rotY);

  stroke (150);
  strokeWeight (5);  
  for (int i=0; i<=loopLength-1; i++) { //draws 3D shape based on text file feed lengths, bend angles, and z bend angles
    line (0, 0, 0, 0, pnt[i].x, 0);       
    translate (0, pnt[i].x, 0);
    rotateZ (-radians(pnt[i].y));
    rotateY (-radians(pnt[i].z));
  }
}

void mousePressed () { 
  mp = !mp;
}
void mouseReleased () { 
  mp = !mp;
}

void keyPressed() { 
  switch (key) {   
  case '1':
    file1();     
    break;
  }
  /*if (keyCode==38) {
   s+=0.01;
   }
   if (keyCode==40) {
   s-=0.01;
   }*/
}

//this subroutine accepts feedback from the arduino. It is not necessary but sometimes needed to establish initial serial communication
/*void serialEvent (Serial p) {
  String inString = arduino.readStringUntil ('\n'); //return signifies new line of commands
  if (inString !=null) {
    println(inString);
  }
}*/


void file1 () { //sends feed and bend angles to the arduino
  println("sending...");
    for (int i = 0; i < loopLength; i++) {
      byte feedmotor = 126; 
      delay (100);
      arduino.write (feedmotor); //send arduino byte marker that signifies feed length
      delay (100);
      arduino.write (byte (int(pnt[i].x))); //send arduino feed legnth
      byte xbend = 125; 
      delay (100);
      arduino.write (xbend); //send arduino byte marker that signifies bend angle
      delay (100);
      arduino.write (byte (int(pnt[i].y))); //send arduino bend angle
      byte zbend = 124; 
      arduino.write (zbend); //send arduino byte marker that signifies z bend angle
      delay (100);
      arduino.write (byte (int(pnt[i].z))); //send arduino z bend angle
    }
  delay (5000);
  byte end = 127;
  println("commands sent to printer");
  arduino.write(end); //send arduino byte marker that signifies end of commands
}
