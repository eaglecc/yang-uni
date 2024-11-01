<template>
	<view>
		<view style="margin: 0; display: block; background-color: #71a419; height: 100vh;">
            <!-- 消除展示框 -->
            <view class="move-list" :style="{
            	width: (size * 7) + 'rpx',
                height: (size + 2) + 'rpx',
            	top: (windowHeight - (size * 3.5)) + 'rpx'
            }">
            </view>
            <!-- 临时展示框 -->
            <view class="temp-list" :style="{
            	width: (size * 7) + 'rpx',
            	top: (windowHeight - (size * 7.5)) + 'rpx'
            }">
                
            </view>
            
            <!-- 功能框 -->
            <view class="function-box" :style="{
                width: (size * 7) + 'rpx',
                height: (size + 2) + 'rpx',
                top: (windowHeight - (size * 3)) + 'rpx'
            }">
                <image src="/static/util/reverse.png" @click="clear(1)" :style="{width: (size - 6) + 'rpx', height: (size - 6) + 'rpx'}"></image>
                <image src="/static/util/up.png" @click="clear(3)" :style="{width: (size - 6) + 'rpx', height: (size - 6) + 'rpx'}"></image>
                <image src="/static/util/clean.png" @click="clear(7)" :style="{width: (size - 6) + 'rpx', height: (size - 6) + 'rpx'}"></image>
            </view>
            
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
		</view>
	</view>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';

const windowHeight = ref(0);
const size = ref(100); // 卡片大小
const rows = ref(1); // 行数
const cols = ref(4); // 列数
const oneGroupCount = ref(3); // 每组卡片数量
const group = ref(8); // 组数
const layerCount = ref(72); // 总层数

// 储存卡片信息和消除状态
const cellHtml = ref([]);
const moveData = ref([]);
const tempData = ref([]);
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
});

function init() {
	moveData.value = [];
    tempData.value = [];
    // 创建一个长度为 oneGroupCount:3 * group:8 * simpleData.length 的数组，用于生成初始卡片组的数量
	const renderData = Array.from(new Array(oneGroupCount.value * group.value))  
		.map(() => simpleData.value.map(v => ({ ...v })))
		.flat()
		.sort(() => Math.random() - 0.5);  // 打乱生成的 renderData 数组
	const cells = [];
	for (let ly = layerCount.value - 1; ly >= 0; ly--) { // ly 表示层级，从最高层往底层排列
		for (let i = 0; i < rows.value; i++) { // 遍历每一行
			for (let j = 0; j < cols.value; j++) { // 遍历每一列
				let item =  renderData.pop();  // 随机决定是否填充卡片数据
                if (ly === layerCount.value - 1 || ly === layerCount.value - 2 || ly === layerCount.value - 3 || ly === layerCount.value - 4 ){
                    if (ly === layerCount.value - 4){
                        cells.push({
                        	ly: layerCount.value - 4,
                        	i: i,
                        	j: j,
                        	left: size.value * j + j * size.value ,  // 计算卡片的左边位置，同一列的j相同，即左边位置相同
                        	top: size.value * i + 180 + (layerCount.value - 1 - ly) * 100,  // 计算卡片的上边位置，同一行的i相同，即顶部位置相同
                        	id: `m${ly}-${i}-${j}`,
                        	name: item.name,
                        	src: item.src,
                        	isMove: false
                        });
                    }else {
                        cells.push({
                        	ly: layerCount.value - 4,
                        	i: i,
                        	j: j,
                        	left: size.value * j + j * size.value ,  // 计算卡片的左边位置，同一列的j相同，即左边位置相同
                        	top: size.value * i + 600 + (layerCount.value - 1 - ly) * 100,  // 计算卡片的上边位置，同一行的i相同，即顶部位置相同
                        	id: `m${ly}-${i}-${j}`,
                        	name: item.name,
                        	src: item.src,
                        	isMove: false
                        });
                    }
                    
                }
                else {
                    cells.push({
                    	ly,
                    	i,
                    	j,
                    	left: size.value * j + j * 100 ,  // 计算卡片的左边位置，同一列的j相同，即左边位置相同
                    	top: size.value * i + ly * 7,  // 计算卡片的上边位置，同一行的i相同，即顶部位置相同
                    	id: `m${ly}-${i}-${j}`,
                    	name: item.name,
                    	src: item.src,
                    	isMove: false
                    });
                }
			}
		}
	}
	cellHtml.value = cells.reverse(); // 反转 cells 数组（确保底层先显示，顶层后显示）
	checkDisabled();
}
// 检查每张卡片是否被遮挡
function checkDisabled() {
	cellHtml.value.forEach((v) => {
		const arr = v.id.substring(1).split('-').map(Number);  // 层级、行、列
		const currentLayer = arr[0]; // 当前层级
		let isBlocked = false; // 默认不被遮挡

		// 遍历比当前层级更高的层
		for (let i = currentLayer + 1; i <= layerCount.value - 4; i++) { 
			// 判断上层的对应行和列是否存在卡片
			if (cellHtml.value.some(item => item.id === `m${i}-${arr[1]}-${arr[2]}`)) {
				isBlocked = true; // 如果找到遮挡，设置为遮挡状态
				break; // 找到遮挡后，退出循环
			}
		}

		v.disabled = isBlocked; // 更新当前卡片的禁用状态
	});
}

function processId(id) {
    // 按 "-" 分割字符串
    let parts = id.split("-");
    
    // 检查最后一部分是否为 "temp"
    if (parts[parts.length - 1] === "temp") {
        // 移除最后一部分
        parts.pop();
        parts.pop();
    }
    
    // 重新组合成字符串并返回
    return parts.join("-");
}


// 点击物品项时，将其移动到一个展示框
function move(item) {
    console.log(item)
    // canMove.value：表示是否可以移动物品项，若为 false 则退出。
    // item.disabled：表示该物品项是否被禁用，若为 true 则退出。
    // item.click：表示物品项是否已经被点击并移动过，若为 true 则退出。
	if (!canMove.value || item.disabled || item.click){
        return; 
    } 
    item.id = processId(item.id)
    // 在 tempData 中查找并移除该元素
    const tempIndex = tempData.value.findIndex(tempItem => {
        return tempItem.id === item.id.replace("-move", "-temp")
    });
    
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
        if (tempIndex > -1) {
            tempData.value.splice(tempIndex, 1);
            // 重新排列 tempData，按每行最多 7 个元素布局
            const maxPerRow = 7;
            tempData.value.forEach((it, index) => {
                const row = Math.floor(index / maxPerRow); // 行数
                const col = index % maxPerRow; // 列数
                const tempList = uni.createSelectorQuery().select(".temp-list");
                tempList.boundingClientRect(data => {
                    let left = data.left;
                    let top = data.top;
                    let scaleLeft = uni.upx2px(left) / left;
                    let scaleTop = uni.upx2px(top) / top;

                    // 根据行和列重新设置元素位置
                    it.left = left / scaleLeft + size.value * col;
                    it.top = top / scaleTop + size.value * row;
                }).exec();
            });
        }

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


// 清除功能
function clear(clearNum) {
    // 获取 tempData 的位置信息
    const tempList = uni.createSelectorQuery().select(".temp-list");
    tempList.boundingClientRect(data => {
        const left = data.left;
        const top = data.top;
        const scaleLeft = uni.upx2px(left) / left;
        const scaleTop = uni.upx2px(top) / top;
        const maxPerRow = 7;  // 每行最多 7 个元素
        const existingCount = tempData.value.length;  // 获取当前 tempData 中已有元素的数量
        if (existingCount + clearNum > 21) {
            uni.showModal({
            	title: '提示',
            	content: '已达到最大道具使用限制！',
            	showCancel: false,
            });
            return;
        }
        // 将每个 moveData 项目移动到 tempData 位置
        moveData.value.slice(0, clearNum).forEach((item, index) => {
            
            const totalIndex = existingCount + index;
            // 计算行和列的位置
            const row = Math.floor(totalIndex / maxPerRow); // 第几行
            const col = totalIndex % maxPerRow;             // 当前行的第几个

            // 设置 left 和 top，使其超过 7 个后换行
            item.left = left / scaleLeft + size.value * col;
            item.top = top / scaleTop + size.value * row;
            item.click = false;
            item.id += "-temp";
            tempData.value.push(item); // 将 item 添加到 tempData
        });

        moveData.value.splice(0, clearNum);
        // 重新排列展示框内的卡片位置
        moveData.value.forEach((it, index) => {
        	const moveList = uni.createSelectorQuery().select(".move-list");
        	moveList.boundingClientRect(data => {
        		let left = data.left
                let scaleLeft = uni.upx2px(left)/left
        		it.left = left/scaleLeft + size.value * index;
        	}).exec();
        });
    }).exec();
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
    display: flex;
    flex-wrap: wrap;
	position: relative;
	border: 1 solid #ddd;
	background-color: rgba(0,0,0,0);
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
