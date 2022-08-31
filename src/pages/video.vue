<template>
    <div>
        <div> <input type="file" accept="image/*" capture="user"> image</div>
        <br>
        <div> <input type="file" accept="video/*" capture="user">video </div>
        <br>
        <div> <input type="file" accept="image/*">no capture </div>
    </div>
    <div class="header">
        <div class="left">
            <div id="startButton" @click="Start" class="button">Start</div>
            <h2>Preview</h2>
            <div class="preview-video">
                <video ref="preview" id="preview" width="100%" height="auto" autoplay muted></video>
            </div>
        </div>

        <div class="right">
            <div class="rightBtn">
                <div id="stopButton" @click="stop" class="button">Stop</div>
            </div>
            <h2>Recording</h2>
            <div class="video"><video ref="recording" id="recording" controls></video></div>
        </div>
    </div>
    <h3>时长：{{ (time / 1000).toFixed(2) }}S</h3>
    <br>
    <a id="downloadButton" ref="downloadButton" class="button">Download</a>
</template>
<script setup>
import { onMounted, onUnmounted, reactive, ref } from "vue";
// import { Decoder, tools, Reader } from 'ts-ebml'

const preview = ref(null)
const recording = ref(null)
const downloadButton = ref(null)

const time = ref(0)
const startTime = ref(0)
let dataChunks = [];
let recorder;

// 开始录制
function startRecording(stream, lengthInMS) {

    recorder = new MediaRecorder(stream, {
        //因为视频和音频分开录制这里直接注释了
        // audioBitsPerSecond: 128000, // 音频码率
        bitsPerSecond: 1843200, // 视频码率
        mimeType: "video/webm", // 编码格式   
    })
    recorder.ondataavailable = (event) => {
        let data = event.data;
        dataChunks.push(data);
    };
    recorder.start(1000);
    console.log(recorder.state + " start to recording .....");
    console.log('time', (new Date().getTime() - startTime.value).toFixed(2))
    startTime.value = new Date().getTime()
}
const stop = async () => {
    // close the recording
    preview.value.srcObject.getTracks().forEach((track) => track.stop());
    recorder.stop();

    time.value = new Date().getTime() - startTime.value
    console.log('datachunks', dataChunks)
    // Play recorded video
    let recordedBlob = new Blob(dataChunks, { type: "video/webm" });
    recording.value.src = URL.createObjectURL(recordedBlob);
    // let file = await getFileFromBlobUrl([recording.value.src])
    console.time('exportVideo')
    getSeekableBlob(recordedBlob, (newBlob) => {
        downloadButton.value.href = URL.createObjectURL(newBlob);
        downloadButton.value.download = `RecordedVideo${new Date().getTime()}.webm`;
    })
    console.timeEnd('exportVideo')
}
const getFileFromBlobUrl = async (blobUrlList) => {
    const pList = blobUrlList.map((url) => {
        return fetch(url)
    })
    const data = await Promise.all(pList)
    const blobList = data.map((v) => v.blob())
    const res = await Promise.all(blobList)
    return res.map(blob => {
        return new File([blob], 'video.webm', { type: 'video/webm' })
    })
}

//访问用户媒体设备的兼容方法
const getUserMedia = (constraints, success, error) => {

    if (navigator.mediaDevices.getUserMedia) {
        //最新的标准API
        navigator.mediaDevices.getUserMedia(constraints).then(success).catch(error);
        var supportedConstraints = navigator.mediaDevices.getSupportedConstraints();
        console.log('supportedConstraints', supportedConstraints)
    } else if (navigator.webkitGetUserMedia) {
        //webkit核心浏览器
        navigator.webkitGetUserMedia(constraints, success, error)
    } else if (navigator.mozGetUserMedia) {
        //firfox浏览器
        navigator.mozGetUserMedia(constraints, success, error);
    } else if (navigator.getUserMedia) {
        //旧版API
        navigator.getUserMedia(constraints, success, error);
    }
}

const success = (stream) => {
    // set the stream to left video
    console.log('srcObject', preview.value)
    const CompatibleURL = window.URL || window.webkitURL;
    //将视频流设置为video元素的源
    try {
        preview.value.src = CompatibleURL.createObjectURL(stream);
    } catch (e) {
        // throw new Error(e);
        preview.value.srcObject = stream;
    }
    console.log('srcObject111', preview.value)
    // preview.srcObject = stream;
    // set the stream to <a> for download
    downloadButton.value.href = stream;
    // captureStream: which is streaming a real-time capture of the content being rendered in the media element. 
    // A MediaStream object  which can be used as a source for audio or video data by other media
    preview.value.captureStream =
        preview.value.captureStream || preview.value.mozCaptureStream;
    startRecording(preview.value.captureStream());
}

const error = (error) => {
    console.log(`访问用户媒体设备失败${error.name}, ${error.message}`);
}

const Start = () => {
    if (navigator.mediaDevices.getUserMedia || navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia) {
        startTime.value = new Date().getTime()
        //调用用户媒体设备, 访问摄像头
        getUserMedia({
            video: {
                width: { ideal: 1280, max: 1280 },
                height: { ideal: 720, max: 720 },
                // facingMode: 'user',
                facingMode: { exact: "environment" },
                frameRate: { ideal: 30, max: 30 },
                torch: true
            }
        }, success, error);
    } else {
        alert('不支持访问用户媒体');
    }
}

// const exportVideo = (videoChunks) => {

//     let blob = new Blob(videoChunks, { "type": "video/webm;codecs=vp8;" });

//     const reader = new Reader()

//     const decoder = new Decoder()

//     const fileReader = new FileReader()

//     fileReader.onload = function () {

//         // 1.解析视频blob

//         const ebmlList = decoder.decode(this.result)

//         ebmlList.forEach(item => reader.read(item))

//         reader.stop()

//         // 2.获取元数据

//         const refinedMetadataBuf = tools.makeMetadataSeekable(

//             reader.metadatas,

//             reader.duration,

//             reader.cues

//         )

//         // 3.获取元数据大小

//         const body = this.result.slice(reader.metadataSize)

//         // 4.写入元数据

//         const newBlob = new Blob([refinedMetadataBuf, body], {
//             type: 'video/webm;'

//         })

//         // 5.将blob转成url
//         const refinedUrl = URL.createObjectURL(newBlob)

//         // 6.自动下载
//         let a = document.createElement('a')
//         a.href = refinedUrl;
//         a.style = "display: none";
//         a.download = `${new Date().toISOString().replace(/T/, ' ').replace(/\..+/, '')}.webm`;
//         a.click();
//     }
//     fileReader.readAsArrayBuffer(blob)
// }

function getSeekableBlob(inputBlob, callback) {
    // EBML.js copyrights goes to: https://github.com/legokichi/ts-ebml
    if (typeof EBML === 'undefined') {
        throw new Error('Please link: https://www.webrtc-experiment.com/EBML.js');
    }

    const reader = new EBML.Reader();
    const decoder = new EBML.Decoder();
    const tools = EBML.tools;

    const fileReader = new FileReader();
    fileReader.onload = function (e) {
        const ebmlElms = decoder.decode(this.result);
        ebmlElms.forEach((element) => {
            reader.read(element);
        });
        reader.stop();
        const refinedMetadataBuf = tools.makeMetadataSeekable(reader.metadatas, reader.duration, reader.cues);
        const body = this.result.slice(reader.metadataSize);

        const newBlob = new Blob([refinedMetadataBuf, body], {
            type: 'video/webm'
        });

        callback(newBlob);
    };
    fileReader.readAsArrayBuffer(inputBlob);
}


</script>
<style scoped>
.header {
    display: flex;
    justify-content: space-around;
}

.button {
    font-size: 24px;
    font-weight: 700;
    color: #646cff;
    padding: 10px 20px;
    margin: 10px;
    border: 1px solid #646cff;
    border-radius: 5px;
}

.preview-video {
    width: 180px;
    height: 400px;
}

#preview {
    width: 180px;
    height: 400px;
}

#recording {
    width: 180px;
    height: 400px;
}
</style>