// Computation in Design
// 
// Andreas Schlegel, 2021.
// 
// uses SVG render library p5.svg.js
// https://github.com/zenozeng/p5.js-svg
//
// Be careful, Performance can slow down if too many 
// elements are rendered.
//
// Read code for more details
// use key s to save canvas to. svg
// use arrow-keys to change rule (each change is reflected in the console)
//

let canvas;
let caColor;
let ca;

function setup() {
  // set canvas renderer to SVG
  canvas = createCanvas(595, 842, SVG); // 592x842 pixels corresponds to A4 210x297mm


  // An initial rule system
  // a rule-system here consists of an array filled 
  // with 8 numbers. arrays are essential components 
  // of a computer program that can store more than 
  // 1 value (like a variable does) but a sequence
  // of values.
  // 
  // @make-changes-here 
  // try to apply a different ruleset here.
  // read the comments above for more details about 
  // the different rulesets and how they are defined.
  // try rule 90 [0,1,0,1,1,0,1,0] 
  // or maybe rule 149 [1,0,0,1,0,1,0,1]
  ruleset = [0, 1, 0, 1, 1, 0, 1, 1]; // this is rule 193

  // @make-changes-here 
  // or use the next line instead of the above.
  ruleset = decimalToBinaryArray(190);
  
  
  caColor = color(171,200,86);

  ca = new CellularAutomata(ruleset);

}

function draw() {


  // If we're done, clear the screen, 
  // (pick a new ruleset and) restart
  // or take any other action here.
  if (ca.finished()) {

    // @make-changes-here
    // (un)comment the 2 code lines below to randomly
    // generate a new ruleset when the current
    // ruleset has reached the bottom of 
    // the canvas
    //
    // ca.randomize();
    // ca.restart();

    // @make-changes-here
    // (un)comment line below to override canvas with bg-color
    //background(bg);
  } else {
    // clearSVG();
    if (ca.generation * ca.scl < height) {
      for (let i = 0; i < int(height / ca.scl); i++) {
        // Generate the next level (the next line)
        ca.generate();
        // Draw the CA
        ca.render();
      }
    }
  }

  // draw the small little rule-label in the 
  // top left corner
  // fill(0)
  // rect(10, 10, 80, 30);
  // fill(255);
  // text("Rule: " + ca.currentRuleDecimal, 20, 29);
}

function keyPressed() {
  switch (keyCode) {
    case (UP_ARROW):
      nextRule(10);
      break;
    case (DOWN_ARROW):
      nextRule(-10);
      break;
    case (LEFT_ARROW):
      nextRule(-1);
      break;
    case (RIGHT_ARROW):
      nextRule(1);
      break;
    case (32): // Space-bar
      nextRule(0); // clear the canvas
      break;
  }
  switch (key) {
    case ('s'):
      save("ca-" + frameCount + ".svg");
      break;
  }

}

function nextRule(theNum) {
  clearSVG();
  ca.next(theNum);
  ca.restart();
}

class CellularAutomata {

  constructor(theRuleSet) {
    this.currentRuleSet = theRuleSet;
    this.currentRuleDecimal = binaryArrayToDecimal(this.currentRuleSet);

    // @make-changes-here 
    // change the size of a single cell-pixel
    this.scl = 38;

    this.cells = new Array(int(width / this.scl));
    this.restart();
  }
  ///////////////////////////////////////////////
  // Set the rules of the CA
  setRuleSet(theRuleSet) {
    this.currentRuleSet = theRuleSet;
  }

  ///////////////////////////////////////////////
  // Make a random ruleset
  randomize() {
    for (let i = 0; i < 8; i++) {
      this.currentRuleSet[i] = int(random(2));
    }
    this.currentRuleDecimal = binaryArrayToDecimal(this.currentRuleSet);
    
  }

  ///////////////////////////////////////////////
  // select the next rule
  next(theNum = 1) {
    this.currentRuleDecimal += theNum;
    if (this.currentRuleDecimal < 0) {
      this.currentRuleDecimal = 255;
    } else if (this.currentRuleDecimal > 255) {
      this.currentRuleDecimal = 0;
    }
    this.currentRuleSet = decimalToBinaryArray(this.currentRuleDecimal);
    console.log("this is rule:", this.currentRuleDecimal, this.currentRuleSet);
  }

  ///////////////////////////////////////////////
  // Reset to generation 0
  restart() {
    for (let i = 0; i < this.cells.length; i++) {
      this.cells[i] = 0;
    }
    // We arbitrarily start with just the 
    // middle cell having a state of "1"
    this.cells[this.cells.length / 2] = 1;
    this.generation = 0;
  }

  ///////////////////////////////////////////////
  // The process of creating 
  // the new generation
  generate() {
    // First we create an empty array 
    // for the new values
    let nextgen = [];

    // For every spot, determine new state by 
    // examing current state, and neighbor states
    // Ignore edges that only have one neighor

    for (let i = 4; i < this.cells.length - 1; i++) {

      // Left neighbor state
      let left = random()<0.7 ? 0:1; //this.cells[i - 1];

      // Current state
      let me = this.cells[i];

      // Right neighbor state
      let right = this.cells[i +3];

      // @make-changes-here
      // mutations:
      // copy and replace code for variables left, me or right
      // after the = sign with the following
      // (of course the modify and play with number)
      // 
      // random()<0.9 ? 0:1; //
      // i%19 == 0 ? 1:this.cells[i + 1]; //


      // Compute next generation state based on ruleset
      nextgen[i] = this.executeRuleSet(left, me, right);
    }


    // Copy the array into current value
    for (let i = 4; i < this.cells.length - 1; i += 1) {
      this.cells[i] = nextgen[i];
    }
    // @make-changes-here
    this.generation += 0.75;
  }

  ///////////////////////////////////////////////
  // This is the easy part, just draw 
  // the cells, fill 255 for '1', fill 0 for '0'
  // 
  // @make-changes-here 
  // change the shapes or colors that will be 
  // rendered per cell.
  // 
  render() {
    noStroke();
    strokeWeight(1);
    for (let i = 0; i < this.cells.length; i++) {

      let x = (i * this.scl);
      let y = this.generation * this.scl;
      let w = 0;
      let h = 0;
      if (y < height) {
        if (this.cells[i] == 1) {
          noFill();
          stroke(caColor);
          
          w = this.scl;
          h = this.scl;
           ellipse(x, y, w, h)
          
          
          // @make-changes-here
          w = this.scl * sin(i * 0.1)*4;
          h = this.scl * sin(i * 0.1)*2;
          ellipse(x, y, w, h)

          // @make-changes-here
          // disable ellipse command above
          // and enable stroke and line below
          // 
          stroke(caColor);
          strokeCap(SQUARE)
          strokeWeight(2);
          // line(x+ this.scl, y, x, y + this.scl);
        } else {
          noFill();
          stroke(caColor);
          
          // @make-changes-here
          let wScl = 0.5; // try a value below 1
          let hScl = 2.5; // try a value below 1
          // rect(x, y, this.scl * wScl, this.scl * hScl);

          // @make-changes-here
          // disable rect command above
          // and enable stroke and line below
          //
          stroke(caColor);
          strokeCap(SQUARE)
          strokeWeight(0.5);
          // line(x, y, x + this.scl, y + this.scl);
        }
      }
    }
  }

  // Implementing the Wolfram rules
  // Could be improved and made more concise, 
  // but here we can explicitly see what is 
  // going on for each case
  executeRuleSet(a, b, c) {
    if (a == 1 && b == 1 && c == 1) {
      return this.currentRuleSet[0];
    }
    if (a == 1 && b == 1 && c == 0) {
      return this.currentRuleSet[1];
    }
    if (a == 1 && b == 0 && c == 1) {
      return this.currentRuleSet[2];
    }
    if (a == 1 && b == 0 && c == 0) {
      return this.currentRuleSet[3];
    }
    if (a == 0 && b == 1 && c == 1) {
      return this.currentRuleSet[4];
    }
    if (a == 0 && b == 1 && c == 0) {
      return this.currentRuleSet[5];
    }
    if (a == 0 && b == 0 && c == 1) {
      return this.currentRuleSet[6];
    }
    if (a == 0 && b == 0 && c == 0) {
      return this.currentRuleSet[7];
    }
    return 0;
  }

  // The CA is done if it reaches the bottom of the screen
  finished() {
    if (this.generation > height / this.scl) {
      return true;
    } else {
      return false;
    }
  }
}

function decimalToBinaryArray(theNum, theLen = 8) {
  const result = new Array(theLen).fill(0);
  let n = theNum;
  let i = 0;
  while (n > 0) {
    result[i] = (n % 2 != 0) ? 1 : 0;
    n = Math.floor(n / 2);
    i += 1;
  }
  return result.reverse();
}

function binaryArrayToDecimal(theArray) {
  return parseInt(theArray.toString().replaceAll(",", ""), 2);
};



// helper to clear the SVG canvas otherwise 
// elements will just keep being added to the 
// initial SVG which results in lagginess and 
// unecessary large file output.
// To avoid this behaviour, clearCanvas.
function clearSVG() {
  canvas.drawingContext.__clearCanvas();
}
