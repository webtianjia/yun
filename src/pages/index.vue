<template>

    <div class="container" :style="{width:windowWidth+'px',height:windowHeight+'px'}">
        <wx-header @iconClick="goTo">
            <div class="span">
                <div class="eye"></div>
            </div>
        </wx-header>
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
                <span>{{faceData.gender > 50 ? "男" : "女"}}</span>
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
                <span>{{faceData.glass ? "Yes" : "No"}}</span>
            </div>
            <div class="face-item">
                <label>颜值：</label>
                <span>{{faceData.beauty}}</span>
            </div>
        </scroll-view>
    </div>
</template>

<script>
    import wxHeader from "../components/header"
    export default {
        name: "index",
        components:{
          wxHeader
        },
        data() {
            return {
                faceData: null,
                fileID: null,
                windowWidth: 0,
                windowHeight: 0,
                desc: "",
                openId: null
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
                        that.fileID = res.fileID
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
                            that.addImg()
                        } else {
                            wx.showToast({title: "请对准人脸哦..", image: "../../../static/imgs/error.png", duration: 3000})
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
            },
            getUserId() {
                const that = this;
                wx.cloud.callFunction({
                    name: 'getUserInfo',
                }).then(res => {
                    that.openId = res.result.openid
                }).catch(err => {
                    console.log(err)
                })
            },
            addImg() {
                const that = this;
                const db = wx.cloud.database();
                db.collection('all_img').add({
                    data: {
                        img_author: {
                            id: that.openId
                        },
                        img_path_id: that.fileID,
                        img_beauty: that.faceData.beauty,
                        img_gender: that.faceData.gender,
                        img_age: that.faceData.age,
                        img_expression: that.faceData.expression,
                        img_desc: that.desc
                    }
                })
            },
            goTo(){
                wx.navigateTo({
                    url: "/pages/list/main"
                });
            }
        },
        created() {
            this.getUserId();
            this.windowWidth = wx.getSystemInfoSync().windowWidth;
            this.windowHeight = wx.getSystemInfoSync().windowHeight;
        }
    }
</script>

<style scoped lang="less">
     .container {
          position: fixed;
          top: 0;
          right: 0;
          bottom: 0;
          left: 0;
          z-index: -1;
          background: data-uri("../static/imgs/bg.jpg") no-repeat center;
          background-size: cover;
      }
    .box {
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        box-shadow: 0 0 10px #fdf;
        padding: 15px;
        box-sizing: border-box;
    }
    .data{
        padding-top:88px ;
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

     .eye{
         width: 20px;
         height: 20px;
         background-color: rgba(255,255,255,0.8);
         border-radius: 50%;
         box-shadow: 30px 0px 0px 0px rgba(255,255,255,0.8);
         position: relative;
         margin: 36px 26px;
     }

     .eye:after{
         background-color: #59488b;
         width: 10px;
         height: 10px;
         box-shadow: 30px 0px 0px 0px #59488b;
         border-radius: 50%;
         left: 9px;
         top: 8px;
         position: absolute;
         content: "";
         animation: eyeball 2s linear infinite alternate;
     }
     @keyframes eyeball{
         0%{left: 9px;}
         100%{left: 1px;}
     }
</style>