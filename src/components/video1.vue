<template>
    <div class="pageContent">
        <div>
            <video id="webcam" :class="cameraVisible ? 'recordSrc' : 'hiddenClass'" muted="true"
                controls="false"></video>
        </div>
        <video class="recordSrc playvideo" controls :src="recordBlob"
            :class="cameraVisible ? 'hiddenClass' : 'recordSrc'"></video>

        <button class="button" @click="startRecording" v-if="step == 0">开始录制</button>
        <button class="button" @click="stopRecording" v-if="step == 1">停止录像</button>
        <button class="button restart" @click="restart" v-if="step == 2">重新录制</button>
        <button class="button upload" @click="uploadVideo" v-if="step == 2">上传视频</button>
    </div>
</template>
 
 
<script>

export default {
    data() {
        return {
            message: "",
            mediaObject: '',
            rec: '',
            chunks: [],
            recordBlob: '',
            step: "0",//0:开始录像, 1:录像中，2：停止录像
            cameraVisible: true, // 显示/隐藏相机
        }
    },

    mounted() {
        this.init();
    },
    components: {


    },
    methods: {
        async init() {
            this.videoContext = uni.createVideoContext('webcam');
            const dom = document.getElementsByClassName("uni-video-video")[0]
            await navigator.mediaDevices.getUserMedia({ video: true, audio: true }).then(function (media) {
                console.log('getUserMedia completed successfully.');
                dom.srcObject = media
            }).catch(function (error) {
                console.log(error.name + ": " + error.message);
                alert(error.name + ": " + error.message)
            });
            console.log(dom.srcObject)
            this.mediaObject = dom.srcObject;
            await this.videoContext.play();
        },

        startRecording() {
            // alert("开始录像了")
            this.step = "1"
            this.rec = new MediaRecorder(this.mediaObject);

            this.chunks = [];
            this.rec.start();
            // alert("启动录像成功")
        },
        stopRecording() {
            this.step = "2";
            this.cameraVisible = false;
            this.rec.stop();
            //alert("停止成功")
            this.rec.ondataavailable = e => this.chunks.push(e.data);
            //alert("导数据了")
            this.rec.onstop = async () => {
                //alert("输出录像blob:"+URL.createObjectURL(new Blob(this.chunks, { type: 'video/mp4' })))
                console.log(URL.createObjectURL(new Blob(this.chunks, { type: 'video/mp4' })))
                this.recordBlob = URL.createObjectURL(new Blob(this.chunks, { type: 'video/mp4' }));
            };
        },
        async restart() {
            this.step = "0";
            this.cameraVisible = true;
            console.log(this.mediaObject)
            // await this.videoContext.play();
        },
        uploadVideo() {
            //关闭摄像头
            if (this.mediaObject) {
                console.log(this.mediaObject);
                console.log(this.mediaObject.getTracks());
                this.mediaObject.getTracks()[0].stop();
                this.mediaObject.getTracks()[1].stop();
            }
            const self = this
            console.log(self.recordBlob)
            //进行下一步处理。。。。。。
        },


    }
}

</script >
   
<style scoped>
uni-page-body {
    height: 100%
}

uni-view {
    display: contents;
}

html,
body {
    margin: 0;
    width: 100%;
    height: 100%
}

body {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.recordSrc {
    width: 100%;
    height: 100%;
    position: absolute;
}

.playvideo {
    left: 0
}

.button {
    position: absolute;
    bottom: 11%;
    left: 50%;
    margin-left: -50px;
    width: 100px;
    border-radius: 42px;
    background-color: red;
}

.restart {
    left: 25%;
}

.upload {
    left: 75%;
}

.hiddenClass {
    visibility: hidden;
}
</style>