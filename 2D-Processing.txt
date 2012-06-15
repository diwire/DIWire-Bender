/* DIWire Bender
 * 3D Wire Bender by Pensa - www.PensaNYC.com
 * Written by Marco Perry. Email DIWire@PensaNYC.com for questions.
 * Drives on 3 Stepper Motors to bender wire in 2D space
 *
 * This file is part of the DIWire project.
 * This file bends in 2D by reading and SVG file (vector graphics file).
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

/* the following program uploads a svg file, draws the 2D shape, allows user to set the print resolution,
   calculates the feed and bend angles to print the 2D shape, and sends these values to the arduino*/

//import these libraries from libraries folder
import processing.serial.*;
import geomerative.*;
Serial arduino;


//Declaring the objects being used to print
RShape shp;
RPoint[] points;
int maxArray = 255;
float[] angles = new float [maxArray];
float[] feeds = new float [maxArray];
float res = 10; //default resolution
int interimSteps = 1; //a counter for the preview
int steps =1;
int lastpty = 0;

boolean chngRes = false;
boolean preview = false;

void setup() {
  size (400, 400); //creates window
  background (255);
  println(Serial.list());
  arduino = new Serial(this, Serial.list() [0], 9600); //sets arduino usb port

  // VERY IMPORTANT: Allways initialize the library in the setup
  RG.init(this);
  shp = RG.loadShape("bendMe.svg"); //loads file from folder
  steps = int(shp.getCurveLength()/res);
  smooth();
  frameRate(24);
  println ();
  println ("SET THE SYSTEM TO HOME POSITION");
  println ("PRESS MOUSE BUTTON TO SET RESOLUTION");
  println ("PRESS:");
  println ("1. PROCESS SHAPE");
  println ("2. PREVIEW SHAPE");
  println ("3. SEND SHAPE TO ARDUINO");
  println ();
  println ("RESOLUTION IS CURRENTLY = " + res);  
  println ("NUMBER OF STEPS = " + steps);
}

void draw() { 
  translate(width/4, height/4);   // Sets draw location and scale
  noFill();

  if (!preview) { //draws exact shape from file bendMe.svg
    RG.setPolygonizer(RG.ADAPTATIVE);
    stroke(0, 0, 200, 150);
    shp.draw();

    RG.setPolygonizer(RG.UNIFORMLENGTH);
    if (chngRes) { //sets resolution if mouse is pressed
      background(255);
      float minDist = shp.getCurveLength()/maxArray; // minimum line distance(resolution) based on array size
      res = map(float(mouseY), 0.0, float(height), minDist, 128); //resolution based on mouse location
      shp.draw();
    }
    RG.setPolygonizerLength(res); 
    points = shp.getPoints(); //point resolution based on mouse location 

    // draw a shape of lines connecting the points
    if (points != null) {
      noFill();
      stroke(0, 200, 0, 50);
      strokeWeight (5);
      beginShape();
      for (int i=0; i<points.length; i++) {
        vertex(points[i].x, points[i].y);
      }
      endShape();
      strokeWeight (1);

      fill(0);
      stroke(0);
      for (int i=0; i<points.length; i++) {
        ellipse(points[i].x, points[i].y, 5, 5);
      }
    }
  }
  else { //draws preview shape based on set resolution
    background (255);
    stroke (random (255), random (255), random (255));
    strokeWeight (5);
    pushMatrix();
    noFill();
    beginShape();
    for (int i=1; i<=steps+2; i++) { //preview bend by relative angles
      line (0, 0, 0, feeds[i-1]);       
      translate (0, feeds [i-1]);
      rotate (-radians(angles [i+1])); //bend
    }
    preview = !preview;  
    popMatrix ();
  }
}


void mousePressed() { //mouse press sets resolution based on mouse location
  println ("RESOLUTION SET TO = " + res + " mm");
  steps = int(shp.getCurveLength()/res);
  println ("NUMBER OF STEPS = " + steps);
  chngRes = !chngRes;
}

void keyPressed() { //If keys are pressed run corresbonding subroutine 
  switch (key) {
  case '1': //calculates bend angles and feed lengths
    calcs();     
    break;

  case '2': //shows a shape preview based on set resolution
    preview = !preview; 
    break;

  case '3': //sends bend angles and feed lengths to the arduino for printing
    sendArd ();
    break;
  }
}


//this subroutine accepts feedback from the arduino. It is not necessary but sometimes needed to establish initial serial communication
/*void serialEvent (Serial p) {
  String inString = arduino.readStringUntil ('\n'); 
  if (inString !=null) {
    println(inString);
  }
}*/


void calcs () {
  float lastAng=0;
  float lastlastAng = 0;

  println ( key );
  println ();
  println ( "Processing shape" );
  println ();
  println (points.length);
  //calculate relative angles between points and assign to array angles[]
  angles [0] = 0; // first angle is zero
  for (int i = 1; i < points.length; i++) {  
    feeds[i-1] = dist (points[i-1].x, points[i-1].y, points[i].x, points[i].y); //feed legths are the distances between points
    float deltaX = points[i].x - points[i-1].x;    //change in x axis between points
    float deltaY = points[i].y - points[i-1].y;    //change in y axis between points

    //the following equations calculate the bend angles between each point
    //there are different equations for each coordinate quadrant
    if ((deltaX >=0 && deltaY >=0)||(deltaX <=0 && deltaY >=0)) { 
      angles[i] =(degrees(atan(deltaX/deltaY)) - (lastAng + lastlastAng));
    }
    if (deltaX <= 0 && deltaY < 0) {
      angles[i] = (180 + abs(degrees(atan(deltaX/deltaY)))) - (lastAng + lastlastAng);
    }
    if (deltaX >0 && deltaY <0) {
      angles[i] = (180 - abs(degrees(atan(deltaX/deltaY)))) - (lastAng + lastlastAng);
    } 
    if (angles[i]>720) {
      angles[i] = angles[i]-360;
    }
    if (angles[i]<-180) {
      angles[i] = 360+angles[i];
    }
    if (angles[i]>180) {
      angles[i] = angles[i]-360;
    }   

    //shows matrix of feeds and bend angles
    println (round(1*(feeds[i-1]))+"\t"+round(angles[i])+"\t"+"0");  //change the 1 before feeds to change scale. z bend remains 0 because this is a 2D shape
    if (abs(angles [i]) > 125) { //beacuse we are sending bytes to the arduino board no angle or feed length can exceed 125
      println ("WARNING - ANGLE TOO BIG");
    }
    lastlastAng = lastAng+lastlastAng; //sets variables to know last 2 bend angles needed to calculate bends
    lastAng = angles[i];
  }
}

void sendArd () { //sends bend angles and feed lengths to the arduino board for printing
  byte feedmotor = 126; //these byte values are markers so the arduino knows if a value is a bend, feed, or end it's the end of the array
  byte bendmotor = 125;
  byte end = 127; 

  println ( key );
  println ();
  println ( "Sending to Arduino" );
  println ( "Printing" );
  println ();

  //send commands
  for (int i = 2; i < steps+233; i++) {

    //sends arduino feed marker followed by feed length
    delay (100);
    arduino.write (feedmotor);
    delay (100);
    arduino.write (byte(round (feeds[i-2])));
    delay (100);
    println (feeds[i-2]);
    //sends arduino bend marker followed by bend angle
    arduino.write (bendmotor);
    delay (100);
    arduino.write (byte(round (-angles[i])));
    println (angles[i]);
  }
  println (end+128); //tells arduino the array of values in complete
  arduino.write(end);
}
