<template>
    <div class='wx-header' :style="{color:color}">
        <div class='icon iconfont' @click="iconClick">
            <slot></slot>
        </div>
        <div class='title'>{{title}}</div>
    </div>
</template>

<script>
    export default {
        name: "header",
        props:{
            color:{
                type:String,
                default:"#fff"
            },
            title:{
                type:String,
                default:""
            }
        },
        methods:{
            iconClick(){
                this.$emit("iconClick")
            }
        }
    }
</script>

<style scoped lang="less">
    .wx-header{
        width: 100%;
        position: fixed;
        z-index: 9999;
        height: 88px;
        .title{
            text-align: center;
        }
        .icon{
            height: 44px;width:44px;font-size: 22px;
        }
    }


</style>