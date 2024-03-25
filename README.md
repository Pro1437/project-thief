# project-thief

// Access the camera.
let stream = null;
navigator.mediaDevices.getUserMedia({ video: true })
.then(function (mediaStream) {
    var video = document.getElementById('videoElement');
    video.srcObject = mediaStream;
    stream = mediaStream;
})
.catch(function (error) {
    console.log("Error accessing the camera: ", error);
});

const loginPage = document.getElementById('loginPage');
const mainPage = document.getElementById('mainPage');
const backButton = document.getElementById('backButton');
const stopButton = document.getElementById('stopButton');

// Handle form submission.
document.getElementById('loginForm').addEventListener('submit', function(event) {
    event.preventDefault();
    // Add your login logic here
    loginPage.style.display = 'none';
    mainPage.style.display = 'block';
});

// Handle back button click
backButton.addEventListener('click', () => {
  mainPage.style.display = 'none';
  loginPage.style.display = 'block';
});

// Handle detect button click.
document.getElementById('detectButton').addEventListener('click', function() {
    // Add your detection logic here
    alert('Detect button clicked!');
});

// Handle stop button click.
stopButton.addEventListener('click', function() {
    if (stream) {
        stream.getTracks().forEach(track => track.stop());
    }
});

// Handle info button click.
document.getElementById('infoButton').addEventListener('click', function() {
    // Add your info logic here
    alert('Info button clicked!');
});

// Send a notification.
function sendNotification(title, options) {
    if (!("Notification" in window)) {
        console.log("This browser does not support desktop notification");
    } else if (Notification.permission === "granted") {
        var notification = new Notification(title, options);
    } else if (Notification.permission !== "denied") {
        Notification.requestPermission().then(function (permission) {
            if (permission === "granted") {
                var notification = new Notification(title, options);
            }
        });
    }
}

// Replace these with your actual model paths
const yoloModelPath = "path/to/yolov3.weights";
const yoloConfigPath = "path/to/yolov3.cfg";
const thiefModelPath = "path/to/thief_detector.weights";
const thiefConfigPath = "path/to/thief_detector.cfg";

// Define the person class ID and thief ID (assuming these exist in your models)
const personClassID = 0; // Assuming person is class 0 in your model
const thiefID = 1; // Assuming thief is class 1 in your model

// Load required libraries (assuming you have them installed with npm or yarn)
const cv2 = require('opencv4js');

async function loadModels() {
  try {
    const model = await cv2.dnn.readNetFromDarknet(yoloConfigPath, yoloModelPath);
    const thiefModel = await cv2.dnn.readNetFromDarknet(thiefConfigPath, thiefModelPath);
    return { model, thiefModel };
  } catch (error) {
    console.error("Error loading models:", error);
    return null;
  }
}

async function detect() {
  if (!videoElement.srcObject) {
    console.error("No video stream available");
    return;
  }

  const cap = new cv2.VideoCapture(videoElement);
  let model, thiefModel;

  try {
    ({ model, thiefModel } = await loadModels());
  } catch (error) {
    console.error("Error loading models:", error);
    return;
  }

  while (true) {
    const { ret, frame } = await cap.read();

    if (!ret) {
      console.error("Error capturing video frame");
      break;
    }

    const height = frame.rows;
    const width = frame.cols;

    // Process the frame for object detection (replace this with your actual logic)
    const blob = cv2.dnn.blobFromImage(frame, 0.00392, (416, 416), (0, 0, 0), true, crop=false);
    model.setInput(blob);
    const outs = model.forward(model.getUnconnectedOutLayersNames());

    let personDetections = [];
    for (const out of outs) {
      for (const detection of out) {
        const scores = detection.slice(5);
        const classID = scores.indexOf(Math.max(...scores));
        if (classID === personClassID) {
          const confidence = scores[classID];
          if (confidence > 0.5) {
            const centerX = Math.round(detection[0] * width);
            const centerY = Math.round(detection[1] * height);
            const w = Math.round(detection[2] * width);
            const h = Math.round(detection[3] * height);
            const x = Math.round(centerX - w / 2);
            const y = Math.round(centerY - h / 2);
            personDetections.push({ x, y, w, h, confidence });
          }
        }
      }
    }

    let thiefDetected = false;
    const thiefBlob = cv2.dnn.blobFromImage(frame, 0.00392, (416, 416), (0, 0, 0), true, crop=false);
    thiefModel.setInput(thiefBlob);
    const thiefOuts = thiefModel.forward(thiefModel.getUnconnectedOutLayersNames());
    for (const thiefOut of thiefOuts) {
      for (const detection of thiefOut) {
        const scores = detection.slice(5);
        const classID = scores.indexOf(Math.max(...scores));
        if (classID === thiefID) {
          const confidence = scores[classID];
          if (confidence > 0.5) {
            thiefDetected = true;
            break;
          }
        }
      }
    }

    // Handle detections (replace this with your logic)
    if (thiefDetected) {
      console.log("Thief detected!");
      // Add your notification logic here (email, phone call, etc.)
    }
}}




<!DOCTYPE html>
<html>
<head>
    <title>Thief Detection System</title>
    <style>
       body {
    font-family: 'Roboto', sans-serif;
    background-color: #b9bfff;
    margin: 0;
    padding: 0;
}

.page {
    background-color: #a69df3;
    width: 300px;
    padding: 20px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border-radius: 16px;
    box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
    transition: box-shadow 0.3s ease, transform 0.3s ease;
}

.page h2 {
    margin: 0;
    padding: 0;
    text-align: center;
    margin-bottom: 20px;
    color: #ffffff;
}

#loginForm label {
    display: block;
    margin-bottom: 10px;
}

#loginForm input[type="text"], #loginForm input[type="password"] {
    width: 100%;
    padding: 10px;
    border: 1px solid #d8bfd8;
    border-radius: 16px;
    margin-bottom: 20px;
    box-sizing: border-box;
    transition: border 0.3s ease;
}

#loginForm input[type="text"]:focus, #loginForm input[type="password"]:focus {
    border: 2px solid #9e7bb5;
}

.login-btn {
    font-weight: bold;
    border-radius: 50px;
    transition: transform 0.3s ease;
}

.login-btn:active {
    transform: scale(0.9);
}

#mainPage {
    width: 100%;
    padding: 20px;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
}

.btn {
    margin-bottom: 20px;
    background-color:#d8bfd8 ;
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 10px;
    border-radius: 16px;
    cursor: pointer;
    box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
    transition: box-shadow 0.3s ease, transform 0.3s ease;
}

.btn:hover {
    background-color: #7b679a;
    transform: scale(1.1);
}

.btn:active {
    transform: scale(0.98);
}

@media (max-width: 600px) {
    .page {
        width: 90%;
    }
}

    </style>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div id="loginPage" class="page">
        <h2>Login</h2>
        <form id="loginForm">
            <label for="username">Username:</label>
            <input type="text" id="username" name="username" required pattern="[a-zA-Z0-9@]+">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password" required pattern="[\w@-]+">
            <input type="submit" value="Login" class="login-btn">
        </form>
    </div>
<div id="mainPage" class="page" style="display: none;">
        <h2>Thief Detection System</h2>
        <button id="detectButton" class="btn">Detect</button>
        <button id="stopButton" class="btn">Stop</button>
        <button id="infoButton" class="btn">Info</button>
        <button id="backButton" class="btn">Back to Login</button>
        <input type="file" id="uploadButton" accept="image/*">
        <div id="video-container">
            <video id="videoElement" autoplay muted></video>
        </div>
    </div>

    <script src="thief.js"></script>
</body>
</html>
