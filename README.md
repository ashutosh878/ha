<!DOCTYPE html> 
<html lang="en"> 

<head> 
    <meta charset="UTF-8" /> 
    <meta name="viewport" content= 
        "width=device-width, initial-scale=1.0" /> 
    <title>3D Text Effect</title> 
</head> 

<body> 
    <h2>INDIAN KILLER ARMY</h2> 
    <style> 
    body { 
        background: green; 
    } 

    h2 { 
        margin: 300px auto; 
        text-align: center; 
        color: white; 
        font-size: 8em; 
        transition: 0.5s; 
        font-family: Arial, Helvetica, sans-serif; 
    } 

    h2:hover { 
        text-shadow: 0 1px 0 #ccc, 0 2px 0 #ccc, 
                    0 3px 0 #ccc, 0 4px 0 #ccc, 
                    0 5px 0 #ccc, 0 6px 0 #ccc, 
                    0 7px 0 #ccc, 0 8px 0 #ccc, 
                    0 9px 0 #ccc, 0 10px 0 #ccc, 
                    0 11px 0 #ccc, 0 12px 0 #ccc, 
                    0 20px 30px rgba(0, 0, 0, 0.5); 
    } 
</style> 
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Face recognition App</title>
    <link rel="stylesheet" href="css/style.css" />
  </head>
  <body>
    <header>Face recognition in the browser using Javascript</header>
    <div class="container">
      <video id="video" height="500" width="500" autoplay muted></video>
    </div>
    <div class="result-container">
      <div id="emotion">Emotion</div>
      <div id="gender">Gender</div>
      <div id="age">Age</div>
    </div>

    <script src="./js/face-api.min.js"></script>
    <script src="./js/main.js"></script>
  </body>
  <script>
      const video = document.getElementById("video");
const isScreenSmall = window.matchMedia("(max-width: 700px)");
let predictedAges = [];

/****Loading the model ****/
Promise.all([
  faceapi.nets.tinyFaceDetector.loadFromUri("/models"),
  faceapi.nets.faceLandmark68Net.loadFromUri("/models"),
  faceapi.nets.faceRecognitionNet.loadFromUri("/models"),
  faceapi.nets.faceExpressionNet.loadFromUri("/models"),
  faceapi.nets.ageGenderNet.loadFromUri("/models")
]).then(startVideo);

function startVideo() {
  navigator.getUserMedia(
    { video: {} },
    stream => (video.srcObject = stream),
    err => console.error(err)
  );
}

/****Fixing the video with based on size size  ****/
function screenResize(isScreenSmall) {
  if (isScreenSmall.matches) {
    video.style.width = "320px";
  } else {
    video.style.width = "500px";
  }
}

screenResize(isScreenSmall);
isScreenSmall.addListener(screenResize);

/****Event Listeiner for the video****/
video.addEventListener("playing", () => {
  const canvas = faceapi.createCanvasFromMedia(video);
  let container = document.querySelector(".container");
  container.append(canvas);

  const displaySize = { width: video.width, height: video.height };
  faceapi.matchDimensions(canvas, displaySize);

  setInterval(async () => {
    const detections = await faceapi
      .detectSingleFace(video, new faceapi.TinyFaceDetectorOptions())
      .withFaceLandmarks()
      .withFaceExpressions()
      .withAgeAndGender();

    const resizedDetections = faceapi.resizeResults(detections, displaySize);
    canvas.getContext("2d").clearRect(0, 0, canvas.width, canvas.height);

    /****Drawing the detection box and landmarkes on canvas****/
    faceapi.draw.drawDetections(canvas, resizedDetections);
    faceapi.draw.drawFaceLandmarks(canvas, resizedDetections);

    /****Setting values to the DOM****/
    if (resizedDetections && Object.keys(resizedDetections).length > 0) {
      const age = resizedDetections.age;
      const interpolatedAge = interpolateAgePredictions(age);
      const gender = resizedDetections.gender;
      const expressions = resizedDetections.expressions;
      const maxValue = Math.max(...Object.values(expressions));
      const emotion = Object.keys(expressions).filter(
        item => expressions[item] === maxValue
      );
      document.getElementById("age").innerText = `Age - ${interpolatedAge}`;
      document.getElementById("gender").innerText = `Gender - ${gender}`;
      document.getElementById("emotion").innerText = `Emotion - ${emotion[0]}`;
    }
  }, 10);
});

function interpolateAgePredictions(age) {
  predictedAges = [age].concat(predictedAges).slice(0, 30);
  const avgPredictedAge =
    predictedAges.reduce((total, a) => total + a) / predictedAges.length;
  return avgPredictedAge;
}

  </script>
  <style>
      body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  height: 100vh;
  background: #2f2f2f;
  width: calc(100% - 33px);
}

canvas {
  position: absolute;
}
.container {
  display: flex;
  width: 100%;
  justify-content: center;
  align-items: center;
}
.result-container {
  display: flex;
  width: 100%;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.result-container > div {
  font-size: 1.3rem;
  padding: 0.5rem;
  margin: 5px 0;
  color: white;
  text-transform: capitalize;
}
#age {
  background: #1e94be;
}
#emotion {
  background: #8a1025;
}
#gender {
  background: #62d8a5;
}
video {
  width: 100%;
}
header {
  background: #42a5f5;
  color: white;
  width: 100%;
  font-size: 2rem;
  padding: 1rem;
  font-size: 2rem;
}
  </style>
</body> 

</html> 

