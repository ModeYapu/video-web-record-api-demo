<!--  -->
<template>
    <div></div>
</template>

<script lang="ts">
export default {
    components: {},
    data() {
        return {
            isRecording: true,
            isBoxShow: false,
            recorder: {},
            videoStream:{}
        };
    },


    computed: {},

    // 初始加载
    mounted() {
        this.isBoxShow = true
        document.title = '视频授权'
        if (!navigator.mediaDevices) {
            alert('您的浏览器不支持获取用户设备');
            return;
        }
    },

    methods: {
        // 开始录制
        startRecord(e: any) {
            this.isRecording = true
            this.isBoxShow = false
            this.recorder.start()
            // this.countDown(true)
        },
        // 结束录制
        stopRecord() {
            this.isRecording = false;
            // this.countDown(false)
            this.videoLength = this.takeTip
            this.isOperateShow = true;
            this.recorder.stop();
            this.stopTracks()
        },
        // 点击上传
        uploadVideo() {
            let file = this.videoStream
            console.log('file', file);
            this.sendFile(file);
            this.fileState = false;
            setTimeout(() => {
                this.fileState = true
            }, 200)
        },
        // 上传文件
        sendFile(file: any) {
            // const token = getToken()
            let that = this
            let xhr = new XMLHttpRequest()
            let fromData = new FormData()
            const newfile = new File([this.file], 'audio/mp4');
            fromData.append('file', newfile, 'file.mp4')


        },
        // 储存录像数据
        saveRecordingData() {
            let blob = new Blob(this.chunks, { 'type': 'video/mp4' })
            console.log('视频大小', (blob.size / (1024 * 1024)).toFixed(2) + 'M');
            this.videoStream = URL.createObjectURL(blob);
            this.file = blob;
            this.onCapture;
            this.chunks = [];
        },
        // 获取录像数据
        getRecordingData(e: any) {
            this.chunks.push(e.data);
        },
        bindEvents() {
            this.recorder.ondataavailable = this.getRecordingData;
            this.recorder.onstop = this.saveRecordingData;
        },
        // 请求多媒体资源
        requestAudioAccess() {
            const constrains = { audio: true, video: true }
            navigator.mediaDevices.getUserMedia({ audio: true, video: true }).then(stream => {
                // var options = {
                // 	mimeType : 'video/mp4'
                // }
                this.recorder = new (window as any).MediaRecorder(stream);
                this.stream = stream;
                let video: any = this.$refs.video
                video.srcObject = this.stream
                video.muted = true
                video.play();
                this.bindEvents();
            }, error => {
                alert('出错，请确保已允许浏览器获取音视频权限');
            });
        },
        // 关闭摄像头
        stopTracks() {
            console.log('stream', this.stream)
            if (this.stream) {
                this.stream.getTracks().forEach((item: any) => {
                    item.stop()
                })
            } else {
                console.log('stream', this.stream)
            }
        }

    }
}

</script>
<style lang='stylus' scoped>
</style>