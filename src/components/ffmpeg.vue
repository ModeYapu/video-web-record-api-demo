<template>
    <h3>Upload a video to transcode to mp4 (x264) and play!</h3>
    <video id="output-video" controls></video><br />
    <input type="file" id="uploader">
    <button @click="cancel()">Cancel</button>
    <p id="message"></p>
</template>
<script setup lang="ts">
import { FFmpeg } from '@ffmpeg/core'
import { onMounted, onUnmounted, reactive } from "vue";

const { createFFmpeg, fetchFile } = FFmpeg;
let ffmpeg = null;
onMounted(() => {


    const transcode = async ({ target: { files } }) => {
        if (ffmpeg === null) {
            ffmpeg = createFFmpeg({ log: true });
        }
        const message = document.getElementById('message');
        const { name } = files[0];
        message.innerHTML = 'Loading ffmpeg-core.js';
        if (!ffmpeg.isLoaded()) {
            await ffmpeg.load();
        }
        ffmpeg.FS('writeFile', name, await fetchFile(files[0]));
        message.innerHTML = 'Start transcoding';
        await ffmpeg.run('-i', name, 'output.mp4');
        message.innerHTML = 'Complete transcoding';
        const data = ffmpeg.FS('readFile', 'output.mp4');

        const video = document.getElementById('output-video');
        video.src = URL.createObjectURL(new Blob([data.buffer], { type: 'video/mp4' }));
    }
    const elm = document.getElementById('uploader');
    elm.addEventListener('change', transcode);


})
const cancel = () => {
    try {
        ffmpeg.exit();
    } catch (e) { }
    ffmpeg = null;
}
</script>
<style scoped>
</style>