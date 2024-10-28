<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
        <button class="game-button" @click="playGames">消除小游戏</button>
	</view>
</template>

<script setup>
import { ref } from 'vue';

function playGames(){
    uni.navigateTo({
        url:"/pages/yang/stage1/stage1"
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
</style>
