<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
        <button class="game-button" @click="playGames">消除小游戏</button>
		<button class="angry-button" @click="noAngry">我不生气了</button>
	</view>
</template>

<script setup>
import { ref } from 'vue';

function playGames(){
    uni.navigateTo({
        url:"/pages/yang/selectStage"
    })
}

function noAngry(){
    uni.navigateTo({
        url:"/pages/noangry/angry/angry"
    })
}

</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
        padding-top: 100rpx;
	}

	.logo {
		height: 200rpx;
		width: 200rpx;
		margin-bottom: 50rpx;
	}

	.game-button {
        font-size: 32rpx;
        color: #ffffff;
        background-color: #42a5f5;
        padding: 0rpx 60rpx;
        border-radius: 50rpx;
        box-shadow: 0 8rpx 16rpx rgba(66, 165, 245, 0.3);
        transition: all 0.3s ease;
    }

    .game-button:hover {
        background-color: #1e88e5;
        box-shadow: 0 12rpx 24rpx rgba(30, 136, 229, 0.4);
        transform: scale(1.05);
    }
	
	.angry-button {
	    font-size: 32rpx;
	    color: #ffffff;
	    background-color: #ff7043;
	    padding: 0rpx 60rpx;
	    border-radius: 50rpx;
	    /* box-shadow: 0 8rpx 16rpx rgba(66, 165, 245, 0.3); */
		box-shadow: 0 8rpx 16rpx rgba(255, 112, 67, 0.3); /* 更新阴影颜色以匹配新的按钮颜色 */
	    transition: all 0.3s ease;
		margin-top: 30rpx; /* 添加上边距创造间隔 */
	}
</style>
