# DIWire Bender

The DIWire Bender is a rapid prototype machine created by [PensaNYC.com](http://pensanyc.com/) that bends metal wire to produce 2D or 3D shapes.

## _What can it be used for?_

Wire models are often needed in design, whether they are for furniture (chair leg scale models) or housewares projects (wire baskets) or even engineering parts (custom springs). But why stop at prototypes? The machine can read any data, why not output artwork from a random number algorithm, or internet data like stock prices and weather stats. You can create mass customized products, like eyeglass frames that fit, or be a street vendor printing jewelry from a person’s silhouette, on demand. And it doesn’t have to be aluminum wire; in principal the machine could bend other materials, including colored electrical wires, some plastics, memory metals, even light pipes to create small light forms. And if you don’t like the output, it could be configured to pass the bent wire through the straightener to start again.

## _How does it work?_

An arduino program imports the bend angles and feed lengths from a processing script. The inputs are in turn translated into DIWire motor commands. During the print, the wire unwinds from a spool, passes through a series of wheels that straighten it, and then feeds through the bending head. The bending head moves in 3 dimensions to create the desired bends and curves.

## _Supported inputs_

- Vector Files: _(e.g., Adobe Illustrator files)_

- 3D Files: _(Rhino or Wavefront OBJ files)_

- Command Text Files: _(e.g., feed 50 mm, bend 90 to right)_

- Coordinates Text Files: _(from 0,0,0 to 0,10,10 to..)_

## _Basic Usage_

1. Upload the program to the Arduino board.

2. Select the desired Processing program, either 2D from svg file, or 3D list of commands.

3. Install libraries.

4. Create either an svg file or txt commands and save it in the corresponding program folder.

5. Connect the arduino board to the computer using the Arduino usb and define the port.

6. Follow the on-screen instructions.

7. Bend!

## _Build Resources_

> ___3D Printed Parts___: STL and STEP files provided.

> ___Bill of Materials___: Excel and Markdown files provided.

> ___Construction Video___: <https://vimeo.com/44274793>

> ___Demo Video___: <http://vimeo.com/43278619>

## _Questions/Comments_

> <DIWire@PensaNYC.com>

## _Active Codebase_

> <http://code.google.com/p/diwire/source/browse/>

## _License_

> See LICENSE.txt

## _Final Thoughts_

> Enjoy and start making something!  