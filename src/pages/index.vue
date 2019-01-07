<template>
    <div>
        <div class="photo-btn" @click="chooseImage"></div>
        <scroll-view v-if="faceData" class="data">
            <div class="box">
                <img mode="aspectFit" :src="faceData.imgPath" lazy-load="true">
            </div>
            <div class="face-item">
                <label>人脸框宽度：</label>
                <span>{{faceData.width}}</span>
            </div>
            <div class="face-item">
                <label>人脸框高度：</label>
                <span>{{faceData.height}}</span>
            </div>
            <div class="face-item">
                <label>性别：</label>
                <span>{{faceData.gender>50?"男":"女"}}</span>
            </div>
            <div class="face-item">
                <label>年龄：</label>
                <span>{{faceData.age}}</span>
            </div>
            <div class="face-item">
                <label>微笑：</label>
                <span>{{faceData.expression}}</span>
            </div>
            <div class="face-item" v-if="faceData.glass">
                <label>是否有眼镜：</label>
                <span>{{faceData.glass?"Yes":"No"}}</span>
            </div>
            <div class="face-item">
                <label>颜值：</label>
                <span>{{faceData.beauty}}</span>
            </div>
        </scroll-view>
        <canvas class="drawingCanvas" canvas-id="drawingCanvas" :style="{width:windowWidth+'px',height:windowHeight+'px'}"></canvas>
    </div>
</template>

<script>
    export default {
        name: "index",
        data() {
            return {
                faceData: null,
                windowWidth: 0,
                windowHeight: 0
            }
        },
        methods: {
            /*1.打开相册或相机*/
            chooseImage() {
                const that = this
                wx.chooseImage({
                    sizeType: ['compressed'],
                    sourceType: ['album', 'camera'],
                    success(response) {
                        const filePath = response.tempFilePaths[0];
                        that.uploadFile(filePath)
                        wx.showLoading({title: "Loading..."})
                    }
                })
            },
            /*2.上传云存储*/
            uploadFile(filePath) {
                const that = this;
                wx.cloud.uploadFile({
                    cloudPath: "img" + new Date().getTime(),
                    filePath: filePath,
                    success: res => {
                        that.getImgPath(res.fileID)
                    }
                })
            },
            /*3.获取云数据库图片*/
            getImgPath(fileID) {
                const that = this;
                wx.cloud.getTempFileURL({
                    fileList: [fileID],
                    success: res => {
                        that.getAuthorization(res.fileList[0].tempFileURL)
                    }
                })
            },
            /*4.调用API人脸识别*/
            getFaceData(imgPath, sign) {
                const that = this;
                wx.request({
                    header: {
                        "host": "recognition.image.myqcloud.com",
                        "content-type": "application/json",
                        "authorization": sign
                    },
                    method: "POST",
                    url: "https://recognition.image.myqcloud.com/face/detect",
                    data: {
                        appid: "1253407082",
                        url: imgPath
                    },
                    success(response) {
                        if (response.data.code === 0) {
                            that.faceData = Object.assign({imgPath}, response.data.data.face[0],);
                        } else {
                            wx.showToast({title: "请对准人脸哦..", image: "../../../static/imgs/error.png"})
                        }
                        wx.hideLoading()
                    }
                })
            },
            /*获取签名*/
            getAuthorization(imgPath) {
                const that = this;
                wx.cloud.callFunction({
                    // 云函数名称
                    name: 'createAuthorization',
                }).then(res => {
                    that.getFaceData(imgPath, res.result.sign)
                }).catch(err => {
                    console.log(err)
                })
            }
        },
        created() {
            wx.cloud.init();
            const windowWidth = wx.getSystemInfoSync().windowWidth;
            const windowHeight = wx.getSystemInfoSync().windowHeight;
            this.windowWidth = windowWidth;
            this.windowHeight = windowHeight;
            drawingCanvas(windowWidth, windowHeight)
        }
    }

    function drawingCanvas(windowWidth, windowHeight) {
        let context = wx.createCanvasContext("drawingCanvas");
        context.drawImage("../static/imgs/bg.jpg", 0, 0);
        context.fill();
        context.draw();
    }
</script>

<style scoped lang="less">
    .drawingCanvas{
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        z-index: -1;
    }
    .data{
       background: transparent;
    }
    body{
        background: transparent;
    }
    .box {
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        box-shadow: 0 0 10px #fdf;
        padding: 15px;
    }

    .photo-btn {
        position: fixed;
        left: 50%;
        margin-left: -45px;
        bottom: 40px;
        width: 90px;
        height: 90px;
        z-index: 999;
        background: data-uri("../static/imgs/photo.png") no-repeat center;
        background-size: cover;
        -webkit-animation-name: scaleDraw; /*关键帧名称*/
        -webkit-animation-timing-function: ease-in-out; /*动画的速度曲线*/
        -webkit-animation-iteration-count: infinite; /*动画播放的次数*/
        -webkit-animation-duration: 5s; /*动画所花费的时间*/
    }

    .face-item {
        display: flex;
        margin-bottom: 10px;
        label {
            flex: 0 0 140px;
            width: 140px;
            text-align: right;
            font-size: 16px;
            color: #666;
        }
        span {
            font-size: 16px;
            color: #add;
        }
    }

    @keyframes scaleDraw { /*定义关键帧、scaleDrew是需要绑定到选择器的关键帧名称*/
        0% {
            transform: scale(1); /*开始为原始大小*/
        }
        25% {
            transform: scale(1.1); /*放大1.1倍*/
        }
        50% {
            transform: scale(1);
        }
        75% {
            transform: scale(1.1);
        }
    }
</style>