// Copyright (c) 2019 ml5
//
// This software is released under the MIT License.
// https://opensource.org/licenses/MIT

/* ===
ml5 Example
PoseNet example using p5.js
=== */

let video;
let poseNet;
let poses = [];

var x, y; // location of the ball
var vx, vy; // speed of the ball

var leftPaddle, rightPaddle;

var PADDLE_WIDTH = 10;
var PADDLE_HEIGHT = 50;
var PADDLE_OFFSET = 30;

var BALL_DIAMETER = 30;

let img;

var txtSize =10;   

let leftScore = 0;
let rightScore = 0;


function preload(){
  img = loadImage('ball.png');
}

function setup() {
  createCanvas(640, 480);
  video = createCapture(VIDEO);
  video.size(width, height);

  // Create a new poseNet method with a single detection
  poseNet = ml5.poseNet(video, modelReady);
  // This sets up an event that fills the global variable "poses"
  // with an array every time new poses are detected
  poseNet.on('pose', function(results) {
    poses = results;
  });
  // Hide the video element, and just show the canvas
  video.hide();
  pongSetup();
}

function modelReady() {
  select('#status').html('Model Loaded');
}

function draw() {
  image(video, 0, 0, width, height);

  
  // We can call both functions to draw all keypoints and the skeletons
  drawKeypoints();
  drawSkeleton();
  
  pongDraw();
}

// A function to draw ellipses over the detected keypoints
function drawKeypoints()  {
  // Loop through all the poses detected
  for (let i = 0; i < poses.length; i++) {
    // For each pose detected, loop through all the keypoints
    let pose = poses[i].pose;
    for (let j = 0; j < pose.keypoints.length; j++) {
      // A keypoint is an object describing a body part (like rightArm or leftShoulder)
      let keypoint = pose.keypoints[j];
      // Only draw an ellipse is the pose probability is bigger than 0.2
      if (keypoint.score > 0.2) {
        fill(255, 0, 0);
        noStroke();
        ellipse(keypoint.position.x, keypoint.position.y, 10, 10);
      }
    }
  }
}

// A function to draw the skeletons
function drawSkeleton() {
  // Loop through all the skeletons detected
  for (let i = 0; i < poses.length; i++) {
    let skeleton = poses[i].skeleton;
    // For every skeleton, loop through all body connections
    for (let j = 0; j < skeleton.length; j++) {
      let partA = skeleton[j][0];
      let partB = skeleton[j][1];
      stroke(255, 0, 0);
      line(partA.position.x, partA.position.y, partB.position.x, partB.position.y);
    }
  }
}

function gameOver(side) {

  x = width/2;
  y = height/2;
  if (side == "left") {
    vx = random(1, 3);
  } else {
    vx = random(-3, -1);
  }
  vy = random(-1, 1);
  
  if (leftScore == 10) {
  text ('you lose!Press r to restart.',width-2*width/10, 60);
  noLoop();
  }
  
  if (rightScore == 10) {
  text ('you lose!Press r to restart.', width/10,60);
  noLoop();
}
}

function pongSetup() {

  x = width/2;
  y = height/2;
  
  vx = 1.5;
  vy = 1;
  
  rightPaddle = height/2;
  leftPaddle = height/2;
}

function pongDraw() {
  background(255,255,255,90);
  
let x1 = width/4;
let x2 = width*3/4;
let y1 = txtSize*5;
  
  stroke(0);
  fill(0);
  textAlign(CENTER);
  textSize(20);
  text('Left Score', x1, txtSize*2);
  text('Right Score', x2, txtSize*2);
  text(leftScore, x1, y1);
  text(rightScore, x2, y1);
  
  stroke(150);
  line (width/2, 0, width/2, height);
  
  fill(0);
  noStroke();
  rectMode(CENTER);
  
 if(poses.length > 0){
      leftPaddle = poses[0].pose.rightEye.y;
   rightPaddle = height-leftPaddle;
  }

  
  rect(PADDLE_OFFSET, leftPaddle, PADDLE_WIDTH, PADDLE_HEIGHT);
  rect(width-PADDLE_OFFSET, rightPaddle, PADDLE_WIDTH, PADDLE_HEIGHT);
  
  // stroke(0);
  image(img, x, y, BALL_DIAMETER, BALL_DIAMETER);
  
  x += vx;
  y += vy;
  
  if (y < 15) {
    vy = -vy;
  }
  if (y > height-15) {
    vy = -vy;
  }
  
  if (x < PADDLE_OFFSET + PADDLE_WIDTH/2 + BALL_DIAMETER/2) {
    if (y < leftPaddle - PADDLE_HEIGHT/2 ||
        y > leftPaddle + PADDLE_HEIGHT/2) {
      gameOver("left");
      rightScore ++;
    } else {
      vx = -vx*5;
     
    }
  }
  if (x > width - (PADDLE_OFFSET + PADDLE_WIDTH/2 + BALL_DIAMETER/2)) {
    if (y < rightPaddle - PADDLE_HEIGHT/2 ||
        y > rightPaddle + PADDLE_HEIGHT/2) {
      gameOver("right"); 
      leftScore ++;
    } else {
      vx = -vx*5;
     }
  }  
}

function keyPressed(){
    
      if(key==='r'){
       rightScore = 0;
        leftScore = 0;
         Loop();
      }
    }
  
