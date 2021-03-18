<!DOCTYPE html>
<html>
<head>
  <script src="face-api.js"></script>
  <script src="js/commons.js"></script>
  <script src="js/faceDetectionControls.js"></script>
  <link rel="stylesheet" href="styles.css">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.100.2/css/materialize.css">
  <script type="text/javascript" src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/0.100.2/js/materialize.min.js"></script>
</head>
<body>
  <div id="navbar"></div>
  <div class="center-content page-container">

    <p> Reference Image: </p>

    <div class="progress" id="loader">
      <div class="indeterminate"></div>
    </div>
    <div style="position: relative" class="margin">
      <img id="refImg" src="" style="max-width: 800px;" />
      <canvas id="refImgOverlay" class="overlay"/>
    </div>

    <div class="row side-by-side">

      <!-- image_selection_control -->
      <div class="row">
        <label>Upload Image:</label>
        <div>
          <input id="refImgUploadInput" type="file" class="bold" onchange="uploadRefImage()" accept=".jpg, .jpeg, .png">
        </div>
      </div>
      <div class="row">
        <label for="refImgUrlInput">Get image from URL:</label>
        <input id="refImgUrlInput" type="text" class="bold">
      </div>
      <button
        class="waves-effect waves-light btn"
        onclick="loadRefImageFromUrl()"
      >
        Ok
      </button>
      <!-- image_selection_control -->

    </div>

    <p> Query Image: </p>

    <div style="position: relative" class="margin">
      <img id="queryImg" src="" style="max-width: 800px;" />
      <canvas id="queryImgOverlay" class="overlay"/>
    </div>


    <div class="row side-by-side">

      <!-- image_selection_control -->
      <div class="row">
        <label>Upload Image:</label>
        <div>
          <input id="queryImgUploadInput" type="file" class="bold" onchange="uploadQueryImage()" accept=".jpg, .jpeg, .png">
        </div>
      </div>
      <div class="row">
        <label for="queryImgUrlInput">Get image from URL:</label>
        <input id="queryImgUrlInput" type="text" class="bold">
      </div>
      <button
        class="waves-effect waves-light btn"
        onclick="loadQueryImageFromUrl()"
      >
        Ok
      </button>
      <!-- image_selection_control -->

    </div>

    <div class="center-content">
      <!-- face_detector_selection_control -->
      <div id="face_detector_selection_control" class="row input-field">
        <select id="selectFaceDetector">
          <option value="ssd_mobilenetv1">SSD Mobilenet V1</option>
          <option value="tiny_face_detector">Tiny Face Detector</option>
        </select>
        <label>Select Face Detector</label>
      </div>
      <!-- face_detector_selection_control -->

      <!-- ssd_mobilenetv1_controls -->
      <span id="ssd_mobilenetv1_controls">
        <div class="row side-by-side">
          <div class="row">
            <label for="minConfidence">Min Confidence:</label>
            <input disabled value="0.5" id="minConfidence" type="text" class="bold">
          </div>
          <button
            class="waves-effect waves-light btn"
            onclick="onDecreaseMinConfidence()"
          >
            <i class="material-icons left">-</i>
          </button>
          <button
            class="waves-effect waves-light btn"
            onclick="onIncreaseMinConfidence()"
          >
            <i class="material-icons left">+</i>
          </button>
        </div>
      </span>
      <!-- ssd_mobilenetv1_controls -->

      <!-- tiny_face_detector_controls -->
      <span id="tiny_face_detector_controls">
        <div class="row side-by-side">
          <div class="row input-field" style="margin-right: 20px;">
            <select id="inputSize">
              <option value="" disabled selected>Input Size:</option>
              <option value="160">160 x 160</option>
              <option value="224">224 x 224</option>
              <option value="320">320 x 320</option>
              <option value="416">416 x 416</option>
              <option value="512">512 x 512</option>
              <option value="608">608 x 608</option>
            </select>
            <label>Input Size</label>
          </div>
          <div class="row">
            <label for="scoreThreshold">Score Threshold:</label>
            <input disabled value="0.5" id="scoreThreshold" type="text" class="bold">
          </div>
          <button
            class="waves-effect waves-light btn"
            onclick="onDecreaseScoreThreshold()"
          >
            <i class="material-icons left">-</i>
          </button>
          <button
            class="waves-effect waves-light btn"
            onclick="onIncreaseScoreThreshold()"
          >
            <i class="material-icons left">+</i>
          </button>
        </div>
      </span>
      <!-- tiny_face_detector_controls -->

    </div>

  </body>

  <script>
    let faceMatcher = null

    async function uploadRefImage(e) {
      const imgFile = $('#refImgUploadInput').get(0).files[0]
      const img = await faceapi.bufferToImage(imgFile)
      $('#refImg').get(0).src = img.src
      updateReferenceImageResults()
    }

    async function loadRefImageFromUrl(url) {
      const img = await requestExternalImage($('#refImgUrlInput').val())
      $('#refImg').get(0).src = img.src
      updateReferenceImageResults()
    }

    async function uploadQueryImage(e) {
      const imgFile = $('#queryImgUploadInput').get(0).files[0]
      const img = await faceapi.bufferToImage(imgFile)
      $('#queryImg').get(0).src = img.src
      updateQueryImageResults()
    }

    async function loadQueryImageFromUrl(url) {
      const img = await requestExternalImage($('#queryImgUrlInput').val())
      $('#queryImg').get(0).src = img.src
      updateQueryImageResults()
    }

    async function updateReferenceImageResults() {
      const inputImgEl = $('#refImg').get(0)
      const canvas = $('#refImgOverlay').get(0)

      const fullFaceDescriptions = await faceapi
        .detectAllFaces(inputImgEl, getFaceDetectorOptions())
        .withFaceLandmarks()
        .withFaceDescriptors()

      if (!fullFaceDescriptions.length) {
        return
      }

      // create FaceMatcher with automatically assigned labels
      // from the detection results for the reference image
      faceMatcher = new faceapi.FaceMatcher(fullFaceDescriptions)

      faceapi.matchDimensions(canvas, inputImgEl)
      // resize detection and landmarks in case displayed image is smaller than
      // original size
      const resizedResults = faceapi.resizeResults(fullFaceDescriptions, inputImgEl)
      // draw boxes with the corresponding label as text
      const labels = faceMatcher.labeledDescriptors
        .map(ld => ld.label)
      resizedResults.forEach(({ detection, descriptor }) => {
        const label = faceMatcher.findBestMatch(descriptor).toString()
        const options = { label }
        const drawBox = new faceapi.draw.DrawBox(detection.box, options)
        drawBox.draw(canvas)
      })
    }

    async function updateQueryImageResults() {
      if (!faceMatcher) {
        return
      }

      const inputImgEl = $('#queryImg').get(0)
      const canvas = $('#queryImgOverlay').get(0)

      const results = await faceapi
        .detectAllFaces(inputImgEl, getFaceDetectorOptions())
        .withFaceLandmarks()
        .withFaceDescriptors()

      faceapi.matchDimensions(canvas, inputImgEl)
      // resize detection and landmarks in case displayed image is smaller than
      // original size
      const resizedResults = faceapi.resizeResults(results, inputImgEl)

      resizedResults.forEach(({ detection, descriptor }) => {
        const label = faceMatcher.findBestMatch(descriptor).toString()
        const options = { label }
        const drawBox = new faceapi.draw.DrawBox(detection.box, options)
        drawBox.draw(canvas)
      })
    }

    async function updateResults() {
      await updateReferenceImageResults()
      await updateQueryImageResults()
    }

    async function run() {
      // load face detection, face landmark model and face recognition models
      await changeFaceDetector(selectedFaceDetector)
      await faceapi.loadFaceLandmarkModel('/')
      await faceapi.loadFaceRecognitionModel('/')
    }

    $(document).ready(function() {
      renderNavBar('#navbar', 'face_recognition')
      initFaceDetectionControls()
      run()
    })
  </script>
</body>
</html>
