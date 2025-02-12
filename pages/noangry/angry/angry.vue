<template>
	<view class="container">
		<!-- 历史记录图标 -->
		<view class="history-icon" @click="toggleDrawer">
			<image class="history-img" src="/static/util/history.png" mode="aspectFit" />
		</view>

		<!-- 侧滑历史记录面板 -->
		<uni-drawer ref="drawerRef" v-if="showDrawer" mode="right"
			:style="{ 'z-index': 9999, position: 'absolute', top: '0', right: '0', height: '100vh' }">
			<view class="history-panel" @click.stop>
				<view class="history-title">历史记录</view>
				<scroll-view class="history-list" scroll-y>
					<view class="history-item" v-for="(item, index) in historyList" :key="index">
						{{ item.time.slice(0, 10) }} - {{ item.records }}
					</view>
				</scroll-view>
			</view>
		</uni-drawer>
		<view v-if="showDrawer" class="drawer-mask" @click="toggleDrawer"></view>

		<!-- 进度显示区域 -->
		<view class="progress-section">
			<text class="percentage">{{ progress.toFixed(1) }}%</text>
			<view class="progress-bar-container">
				<view class="progress-bar" :style="{ width: `${Math.max(0, progress)}%` }"></view>
			</view>
		</view>

		<!-- 按钮控制区域 -->
		<view class="button-group">
			<button class="control-btn" @click="changeProgress(10)">+10%</button>
			<button class="control-btn" @click="changeProgress(20)">+20%</button>
			<button class="control-btn" @click="changeProgress(50)">+50%</button>
			<button class="control-btn" @click="changeProgress(80)">+80%</button>
			<button class="control-btn minus" @click="changeProgress(-1)">-1%</button>
		</view>

		<!-- 图片区域 -->
		<view class="image-container">
			<image class="gif-image" src="/static/angry/cat.gif" mode="aspectFit" />
		</view>

		<!-- 标题框 -->
		<view class="title-box">
			<input class="title-input" v-model="titleText" placeholder="请输入标题" />
		</view>

		<!-- 重置按钮 -->
		<view class="reset-btn-container">
			<button class="control-btn reset" @click="resetProgress">我好了，下回再说</button>
		</view>
	</view>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const progress = ref(0)
const historyList = ref([])
const showDrawer = ref(false)  // 新增控制抽屉显示的状态
const titleText = ref('') 

onMounted(() => {
	fetchHistory()
})

// 获取历史记录数据
const fetchHistory = async () => {
	try {
		const res = await uni.request({
			url: 'http://117.72.91.96:8501/api/v1/xutils/getAngryHistory',
			method: 'GET'
		})
		// console.log("res...",res.data)
		if (res.statusCode === 200) {
			// 转换数据格式
			historyList.value = res.data.data.map(item => ({
				time: item.time,
				records: item.records
			}))
		}
	} catch (error) {
		showModal("获取历史记录失败！")
	}
}

// 切换抽屉状态
const toggleDrawer = () => {
	showDrawer.value = !showDrawer.value
}

const changeProgress = (value) => {
	let res = progress.value + value
	if (res >= 100) {
		showModal("✌，哄好了！")
		progress.value = 100
	} else if (res < 0) {
		showModal("别减了，再减没了！")
		progress.value = res
	} else {
		progress.value = res
	}

}

const recordHistory = async () => {
	const record = {
		progress: progress.value.toFixed(1),
		records: titleText.value
	}
	try {
		const res = await uni.request({
			url: 'http://117.72.91.96:8501/api/v1/xutils/addAngryHistory',
			method: 'POST',
			data: record,
			header: {
				'Content-Type': 'application/json'
			}
		})

		if (res.statusCode === 200) {
			fetchHistory()
		} else {
			showModal("保存记录请求处理失败！")
		}
	} catch (error) {
		showModal("记录保存失败！")
	}
}

const resetProgress = () => {
	recordHistory() //记录历史
	progress.value = 0
	titleText.value = ''
	showModal("已记录")
}

function showModal(contents) {
	uni.showModal({
		title: '提示',
		content: contents,
		showCancel: false,
	});
}
</script>

<style>
.container {
	display: flex;
	flex-direction: column;
	align-items: center;
	min-height: 100vh;
	padding: 20px;
	background-color: #f5f5f5;
}

.title-box {
	width: 100%;
	text-align: center;
	padding: 10px 0;  /* 减小padding */
	background-color: #ffffff;
	box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	margin: 10px 0;  /* 调整margin */
}

.title-input {
	font-size: 20px;  /* 减小字体大小 */
	font-weight: bold;
	color: #333;
	text-align: center;
	border: none;
	outline: none;
	width: 100%;
	max-width: 500px;
	background-color: transparent;
}

.history-icon {
	position: absolute;
	top: 0px;
	right: 0px;
	z-index: 999;
	cursor: pointer;
	width: 40px;
	height: 40px;
}

.history-img {
	width: 100%;
	/* 使用100%填充容器 */
	height: 100%;
	/* 使用100%填充容器 */
}

.drawer-mask {
	position: fixed;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background-color: rgba(0, 0, 0, 0.3);
	z-index: 9998;
}

.history-panel {
	padding: 20px;
	width: 250px;
	background-color: #ffffff;
}

.history-title {
	font-size: 20px;
	font-weight: bold;
	margin-bottom: 20px;
}

.history-list {
	height: calc(100vh - 60px);
}

.history-item {
	padding: 10px;
	border-bottom: 1px solid #eee;
}

.progress-section {
	width: 100%;
	max-width: 500px;
	margin-bottom: 20px;
}

.percentage {
	text-align: center;
	font-size: 24px;
	margin-bottom: 10px;
	color: #333;
}

.progress-bar-container {
	width: 100%;
	height: 20px;
	background-color: #ddd;
	border-radius: 10px;
	overflow: hidden;
}

.progress-bar {
	height: 100%;
	background-color: #4CAF50;
	transition: width 0.3s ease;
}

.image-container {
	width: 100%;
	display: flex;
	justify-content: center;
	align-items: center;
	margin-top: 20px;
}

.gif-image {
	width: 100%;
	max-width: 300px;
	height: 300px;
}

.button-group {
	display: flex;
	flex-wrap: wrap;
	gap: 10px;
	justify-content: center;
	width: 100%;
	max-width: 500px;
}

.control-btn {
	padding: 10px 20px;
	border: none;
	border-radius: 5px;
	background-color: #4CAF50;
	color: white;
	cursor: pointer;
	transition: background-color 0.3s;
}

.control-btn:hover {
	background-color: #45a049;
}

.control-btn.minus {
	background-color: #f44336;
}

.control-btn.minus:hover {
	background-color: #da190b;
}

.reset-btn-container {
	margin-top: 20px;
	width: 100%;
	display: flex;
	justify-content: center;
}

.control-btn.reset {
	background-color: #ff9800;
}

.control-btn.reset:hover {
	background-color: #f57c00;
}
</style>