<template>
    <div class="container">
        <ul class="list">
            <li class="list-item" style="height: 100px"></li>
            <li class="list-item"></li>
            <li class="list-item" style="height: 100px"></li>
            <li class="list-item"></li>
            <li class="list-item"></li>

            <li class="list-item"></li>
            <li class="list-item" style="height: 100px"></li>
            <li class="list-item"></li>
            <li class="list-item"></li>
            <li class="list-item"></li>
            <li class="list-item"></li>
            <li class="list-item"></li>
            <li class="list-item" style="height: 100px"></li>
            <li class="list-item"></li>
            <li class="list-item"></li>
        </ul>
    </div>
</template>

<script>
    export default {
        name: "faceList"
    }
</script>

<style scoped lang="less">
    .container {
        width: 100%;
        min-height: 100vh;
        background-image: linear-gradient(to top, #a8edea 0%, #fed6e3 100%);
    }

    .list {
        display: flex;
        flex-flow: row wrap;
        align-content: flex-start;
        box-sizing: border-box;
        &:after{
            content: '';
            flex-grow: 99999; //使用伪类填充容器，防止出现最后一张图片单独占一行的情况
        }
        .list-item {
            box-sizing: border-box;
            background-color: white;
            flex: 0 0 46%;
            height: 50px;
            border: 1px dotted grey;
            list-style: none;
            margin: 10px 2% 10px 2%;
            &:nth-child(2n){
                margin-right: 0;
            }
        }
    }
</style>