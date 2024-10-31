<template>
	<view>
		<view style="margin: 0; display: block; background-color: #71a419; height: 100vh;">
			<!-- 卡牌框 -->
			<view class="mall">
				<view v-for="(item, index) in cellHtml" :key="index" class="item" :class="{ disabled: item.disabled }"
					@click="move(item)" :id="item.id" :style="{
						width: size + 'rpx',
						height: size + 'rpx',
						left: item.left + 'rpx',
						top: item.top + 'rpx'
					}">
					<image :style="{
						width: (size - 6) + 'rpx',
						height: (size - 6) + 'rpx',
						borderRadius: '8rpx',
						backgroundColor: '#ffffff',
						border: '2rpx solid #71a419',
						borderBottom: '10rpx solid #71a419'
					}" :src="item.src"></image>
				</view>
			</view>
            <!-- 临时展示框 -->
            <view class="temp-list" :style="{
            	width: (size * 7) + 'rpx',
                height: (size + 2) + 'rpx',
            	top: (windowHeight - (size * 6)) + 'rpx'
            }">
            </view>
            
            <!-- 消除展示框 -->
            <view class="move-list" :style="{
            	width: (size * 7) + 'rpx',
                height: (size + 2) + 'rpx',
            	top: (windowHeight - (size * 3.5)) + 'rpx'
            }">
            </view>

            <!-- 功能框 -->
            <view class="function-box" :style="{
                width: (size * 7) + 'rpx',
                height: (size + 2) + 'rpx',
                top: (windowHeight - (size * 3)) + 'rpx'
            }">
                <image src="/static/util/reverse.png" @click="undo" :style="{width: (size - 6) + 'rpx', height: (size - 6) + 'rpx'}"></image>
                <image src="/static/util/up.png" @click="moveUp" :style="{width: (size - 6) + 'rpx', height: (size - 6) + 'rpx'}"></image>
                <image src="/static/util/clean.png" @click="clear" :style="{width: (size - 6) + 'rpx', height: (size - 6) + 'rpx'}"></image>
            </view>
		</view>
	</view>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';

const windowHeight = ref(0);
const size = ref(100); // 卡片大小
const rows = ref(7); // 行数
const cols = ref(7); // 列数
const oneGroupCount = ref(3); // 每组卡片数量
const group = ref(8); // 组数
const layerCount = ref(24); // 总层数

// 储存卡片信息和消除状态
const cellHtml = ref([]);
const moveData = ref([]);
const canMove = ref(true);

// 卡片内容
const simpleData = ref([
	{ name: '火把', src: '/static/img/1.jpg' },
	{ name: '毛球', src: '/static/img/2.jpg' },
	{ name: '羊毛', src: '/static/img/3.jpg' },
	{ name: '草堆', src: '/static/img/4.jpg' },
	{ name: '白菜', src: '/static/img/5.jpg' },
	{ name: '牛奶', src: '/static/img/6.jpg' },
	{ name: '胡萝卜', src: '/static/img/7.jpg' },
	{ name: '玉米', src: '/static/img/8.jpg' },
	{ name: '刷子', src: '/static/img/9.jpg' },
	{ name: '鸡腿', src: '/static/img/chicken.png' },
    { name: '鸡蛋', src: '/static/img/egg.png' },
    { name: '鱼', src: '/static/img/fish.png'},
    { name: '汉堡', src: '/static/img/hamburger.png'},
    { name: '牛奶', src: '/static/img/milk.png' },
    { name: '面条', src: '/static/img/noodles.png' },
]);

// 页面初始化
onMounted(() => {
	init();
	const height = uni.getSystemInfoSync().windowHeight;
	const top = uni.upx2px(height) / height;
	windowHeight.value = height / top;
    console.log("windowHeight:....",windowHeight.value)
});

function init() {
	moveData.value = [];
    // 创建一个长度为 oneGroupCount:3 * group:8 * simpleData.length 的数组，用于生成初始卡片组的数量
	const renderData = Array.from(new Array(oneGroupCount.value * group.value))  
		.map(() => simpleData.value.map(v => ({ ...v })))
		.flat()
		.sort(() => Math.random() - 0.5);  // 打乱生成的 renderData 数组
	const cells = [];
	for (let ly = layerCount.value - 1; ly >= 0; ly--) { // ly 表示层级，从最高层往底层排列
		for (let i = 0; i < rows.value; i++) { // 遍历每一行
			for (let j = 0; j < cols.value; j++) { // 遍历每一列
				let pyStep = (ly + 1) % 2 === 0 ? size.value / 2 : 0; //pyStep 决定层级的偏移步长，使得偶数层相对奇数层偏移 size.value / 2
				let item = Math.random() > 0.7 && renderData.pop();  // 随机决定是否填充卡片数据
				item && cells.push({
					ly,
					i,
					j,
					left: size.value * j + pyStep,  // 计算卡片的左边位置，同一列的j相同，即左边位置相同
					top: size.value * i + pyStep,  // 计算卡片的上边位置，同一行的i相同，即顶部位置相同
					id: `m${ly}-${i}-${j}`,
					name: item.name,
					src: item.src,
					isMove: false
				});
			}
		}
	}
	cellHtml.value = cells.reverse(); // 反转 cells 数组（确保底层先显示，顶层后显示）
	checkDisabled();
    console.log(cellHtml)
}

// 检查每张卡片是否被遮挡
function checkDisabled() {
	cellHtml.value.forEach((v) => {
		const arr = v.id.substring(1).split('-').map(Number);  // 层级、行、列
		const isPy = (arr[0] + 1) % 2 === 0;
		for (let i = arr[0] + 1; i <= layerCount.value - 1; i++) { // 遍历比当前层级更高的层
			const isPyB = (i + 1) % 2 === 0; // 判断上层的奇偶性
			const hasOverlay = checkOverlay(arr, i, isPy, isPyB);
			v.disabled = hasOverlay;
			if (hasOverlay) break;
		}
	});
}

function checkOverlay(arr, i, isPy, isPyB) {
	if (isPy === isPyB) {
		return !!cellHtml.value.find(item => item.id === `m${i}-${arr[1]}-${arr[2]}`);
	} else if (isPy && !isPyB) {
		const ids = [
			`${i}-${arr[1]}-${arr[2]}`,
			`${i}-${arr[1]}-${arr[2] + 1}`,
			`${i}-${arr[1] + 1}-${arr[2]}`,
			`${i}-${arr[1] + 1}-${arr[2] + 1}`
		];
		return ids.some(k => cellHtml.value.find(item => item.id === `m${k}`));
	} else {
		const ids = [
			`${i}-${arr[1]}-${arr[2]}`,
			`${i}-${arr[1]}-${arr[2] - 1}`,
			`${i}-${arr[1] - 1}-${arr[2]}`,
			`${i}-${arr[1] - 1}-${arr[2] - 1}`
		];
		return ids.some(k => cellHtml.value.find(item => item.id === `m${k}`));
	}
}

// 点击物品项时，将其移动到一个展示框
function move(item) {
    // canMove.value：表示是否可以移动物品项，若为 false 则退出。
    // item.disabled：表示该物品项是否被禁用，若为 true 则退出。
    // item.click：表示物品项是否已经被点击并移动过，若为 true 则退出。
	if (!canMove.value || item.disabled || item.click){
        return; 
    } 
    
	canMove.value = false; // 在本次操作完成前不允许再次移动其他物品
	const moveList = uni.createSelectorQuery().select(".move-list");
	moveList.boundingClientRect(data => {
		let left = data.left
		let top = data.top
        let scaleLeft = uni.upx2px(left)/left
        let scaleTop = uni.upx2px(top)/top
		item.left = left/scaleLeft + size.value * moveData.value.length;
		item.top = top/scaleTop;
		item.click = true;  // 标记物品项为已点击状态，防止重复点击
		item.id += "-move";  // 更新物品项 id，标识其已移动
        
		moveData.value.push(item);
		modeEnd(item);
		canMove.value = true;
	}).exec();
}

function modeEnd(item) {
	// 获取所有名称相同的项目
	let findResult = moveData.value.filter(v => v.name === item.name);

	if (findResult.length === 3) {
		setTimeout(() => {
			findResult.forEach(v => {
				const el = cellHtml.value.find(i => i.id === v.id);
				if (el) {
					// 设置左侧位置为负数，并标记为已移动
					el.left = -size.value;
					el.isMove = true;
				}
				// 从 moveData 中删除匹配项
				const index = moveData.value.findIndex(i => i.id === v.id);
				if (index > -1) {
					moveData.value.splice(index, 1);
				}
			});
			// 重新排列展示框内的卡片位置
			moveData.value.forEach((it, index) => {
				const moveList = uni.createSelectorQuery().select(".move-list");
				moveList.boundingClientRect(data => {
					let left = data.left
                    let scaleLeft = uni.upx2px(left)/left
					it.left = left/scaleLeft + size.value * index;
				}).exec();
			});
		}, 300);
	}

	setTimeout(() => {
		// 游戏结束或通关提示
		if (moveData.value.length === 7) {
            showModal("池子已满，游戏结束")
		} else if (cellHtml.value.every(item => item.isMove)) {
            showModal("恭喜通关")
		}
	}, 300);

	// 检查卡片是否被覆盖
	checkDisabled();
}


function showModal(contents) {
	uni.showModal({
		title: '提示',
		content: contents,
		showCancel: false,
		success: () => init()
	});
}
</script>

<style scoped>
.main {
	position: relative;
}

.item {
	position: absolute;
	color: #fff;
	display: flex;
	justify-content: center;
	align-items: center;
	border-radius: 10rpx;
	transition: left .3s, top .3s;
}

.item:after {
	content: '';
	position: absolute;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	transition: background-color .3s;
}

.disabled:after {
	background-color: rgba(0, 0, 0, 0.7);
}

.move-list {
	position: relative;
	border: 1 solid #ddd;
	background-color: rgb(150, 91, 27);
	margin: 0 auto;
}

.temp-list {
	position: relative;
	border: 1 solid #ddd;
	background-color: rgb(150, 91, 27);
	margin: 0 auto;
}

.function-box {
    position: relative;
    border: 1 solid #ddd;
    background-color: rgba(0,0,0,0);
    margin: 0 auto;
    display: flex;
    justify-content: space-around;
    align-items: center;
}
</style>
