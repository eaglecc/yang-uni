<template>
    <view class="container">
        <view class="header">
            <text class="title">选择关卡</text>
        </view>
        <view class="levels">
            <button v-for="(level, index) in levels" :key="index" :class="['level-button', `level-${index + 1}`]"
                @click="playGames(level)">
                {{ level.name }}
            </button>
        </view>
    </view>
</template>

<script setup>
    import { reactive, ref, onMounted } from 'vue';

    const levels = ref([{
            name: '关卡 1',
            path: '/pages/yang/stage/stage1'
        },
        {
            name: '关卡 2',
            path: '/pages/yang/stage/stage2'
        },
        {
            name: '暂未开通',
            path: '/pages/yang/selectStage'
        },
        // {
        //     name: '暂未开通',
        //     path: '/pages/yang/selectStage'
        // },
        // {
        //     name: '暂未开通',
        //     path: '/pages/yang/selectStage'
        // },
    ]);

    function playGames(level){
        uni.navigateTo({
            url: level.path,
        })
    }
    
</script>

<style scoped>
    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100vh;
        background-color: #f5f5f5;
    }

    .header {
        margin-bottom: 40px;
    }

    .title {
        font-size: 28px;
        font-weight: bold;
        color: #333;
    }

    .levels {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
    }

    .level-button {
        background-color: #007AFF;
        color: #fff;
        border: none;
        border-radius: 8px;
        padding: 20px 30px;
        margin: 10px;
        font-size: 18px;
        cursor: pointer;
        transition: background-color 0.3s;
    }

    .level-1 {
        background-color: #FF5733;
    }

    .level-2 {
        background-color: #33FF57;
    }

    .level-3 {
        background-color: #3357FF;
    }

    .level-4 {
        background-color: #F3FF33;
    }

    .level-5 {
        background-color: #FF33A1;
    }
</style>