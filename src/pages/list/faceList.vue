<template>
    <div class="container">
        <wx-header>

        </wx-header>
        <scroll-view>
            <ul class="list">
                <li v-for="item in list" :key="item.id" class="list-item">
                    <div class="card">
                        <img mode="widthFix" :src="item.tempFileURL" class="user-img">
                        <span class="beauty">颜值{{item.img_beauty}}</span>
                        <div class="text">
                            <span>性别：{{item.img_gender|sex}}</span>
                            <span>年龄：{{item.img_age}} 岁</span>
                        </div>
                    </div>
                </li>
            </ul>
        </scroll-view>
    </div>
</template>

<script>
    import wxHeader from "../../components/header"

    export default {
        name: "faceList",
        components: {
            wxHeader
        },
        data() {
            return {
                list: [],
                param: {
                    pageSize: 5,
                    pageIndex: 1,
                },
                total: 0
            }
        },
        computed: {
            isNoDataBottom: () => {
                if (this.total) {
                    return (parseInt(this.total / this.param.pageSize) + 1) <= this.param.pageIndex;
                }
            }
        },
        methods: {
            getImgPath(list) {
                let fileIds = [];
                list.forEach(item => {
                    fileIds.push(item.img_path_id)
                });
                const that = this;
                wx.cloud.getTempFileURL({
                    fileList: fileIds,
                    success: res => {
                        for (let i = 0; i < list.length; i++) {
                            if (list[i].img_path_id === res.fileList[i].fileID) {
                                list[i].tempFileURL = res.fileList[i].tempFileURL
                            }
                        }
                        that.list.push(list)
                    }
                })
            },
            getAllImg() {
                const that = this;
                wx.cloud.callFunction({
                    name: 'paginator',
                    data: {
                        "dbName": "all_img",
                        "pageIndex": that.param.pageIndex,
                        "pageSize": that.param.pageSize
                    }
                }).then(res => {
                    that.total = res.result.total;
                    that.getImgPath(res.result.data)
                }).catch(err => {
                    console.log(err)
                })
            },
        },
        filters: {
            sex(value) {
                let text;
                if (value <= 49) {
                    text = "女";
                } else if (value >= 50 && value <= 51) {
                    text = "中性";
                } else {
                    text = "男";
                }
                return text
            }
        },
        created() {
            this.getAllImg()
        },
        onReachBottom() {
            if (this.isNoDataBottom) return;
            this.param.pageIndex++;
            this.getAllImg()
        }
    }

</script>

<style scoped lang="less">
    .container {
        width: 100%;
        min-height: 100vh;
        background-image: linear-gradient(to top, #a8edea 0%, #fed6e3 100%);
    }

    .list {
        padding: 10px;
        padding-top: 88px;
        column-count: 2;
        column-gap: 10px;
    }

    .card {
        height: 100%;
        overflow: auto;
        border-radius: 3px;
        padding: 2%;
        margin-bottom: 10px;
        box-sizing: border-box;
        background-image: linear-gradient(to right, #74ebd5 0%, #9face6 100%);
        position: relative;
        .user-img {
            width: 100%;
            box-shadow: 0 0 5px #d00;
        }
        .text {
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: #fff;
            font-size: 14px;
            padding: 0 5px;
        }
        .beauty {
            position: absolute;
            right: 5px;
            padding: 4px;
            border-radius: 3px;
            box-shadow: 0 0 5px #fff;
            top: 5px;
            color: #07f;
            font-size: 14px;
            background-image: linear-gradient(to top, #fad0c4 0%, #fad0c4 1%, #ffd1ff 100%);;
        }
    }

</style>