```html
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.1.2/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/pose@0.6/dist/teachablemachine-pose.min.js"></script>
<script type="text/javascript">
    // Full API: https://github.com/googlecreativelab/teachablemachine-libraries

    // the json file (model topology) has a reference to the bin file (model weights)
    const checkpointURL = '{{MODEL_URL}}';
    // the metatadata json file contains the text labels of your model and additional information
    const metadataURL = '{{METADATA_URL}}';

    let model; let webcamEl; let ctx; let maxPredictions;

    async function init() {
        // load the model and metadata
        model = await tmPose.load(checkpointURL, metadataURL);
        maxPredictions = model.getTotalClasses();

        const width = 200; const height = 200;

        // optional function for creating a webcam
        // webcam has a square ratio and is flipped by default to match training
        webcamEl = await tmPose.getWebcam(width, height);
        webcamEl.play();
        // document.body.appendChild(webcamEl);

        // optional function for creating a canvas to draw the webcam + keypoints to
        const flip = true;
        const canvas = tmPose.createCanvas(width, height, flip);
        ctx = canvas.getContext('2d');
        document.body.appendChild(canvas);

        window.requestAnimationFrame(loop); // kick of pose prediction loop
    }

    async function loop(timestamp) {
        await predict();
        window.requestAnimationFrame(loop);
    }

    async function predict() {
        // Prediction #1: run input through posenet
        // estimatePose can take in an image, video or canvas html element
        const flipHorizontal = false;
        const { pose, posenetOutput } = await model.estimatePose(webcamEl, flipHorizontal);
        // Prediction 2: run input through teachable machine assification model
        const prediction = await model.predict(posenetOutput, flipHorizontal, maxPredictions);

        ctx.drawImage(webcamEl, 0, 0);
        // draw the keypoints and skeleton
        if (pose) {
            const minPartConfidence = 0.5;
            tmPose.drawKeypoints(pose.keypoints, minPartConfidence, ctx);
            tmPose.drawSkeleton(pose.keypoints, minPartConfidence, ctx);
        }

        console.log(prediction);
    }

    init();
</script>
```