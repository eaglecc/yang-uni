<template>
    <view class="box">
        <!-- top -->
        <view class="top-box">
            <view class="card-box">
                <view class="card" @click="selectCard(item, index, `image-${index}`)" v-for="(item, index) in allCardList"
                    :key="index" :style="{ left: item.left, top: item.top }">
                    <image :id="`image-${index}`" :src="`/static/img/${item.icon}.png`" alt="icon" style="width: 20px; height: 20px;" ></image>
                </view>
            </view>
        </view>
        <!-- bottom -->
        <view class="bottom-box">
            <view class="bottom-selected-box">
                <view class="selected-card" v-for="(item, index) in selectedCard" :key="index">
                    <image :src="`/static/img/${item.icon}.png`" alt="icon" style="width: 20px; height: 20px;" ></image>
                </view>
            </view>
        </view>
    </view>
</template>

<script setup>
import { onMounted, ref } from 'vue'

const allCardList = ref([])
const selectedCard = ref([])
const currentSelectCard = ref()
const receivedCardList = ref([])

const iconList = ref([
    { picIndex: 1, icon: "chicken" },
    { picIndex: 2, icon: "egg" },
    { picIndex: 3, icon: "fish" },
    { picIndex: 4, icon: "hamburger" },
    { picIndex: 5, icon: "milk" },
    { picIndex: 6, icon: "noodles" },
    { picIndex: 7, icon: "radish" }
])

// 初始化receivedCardList，确保数据个数是3的倍数
const initializeReceivedCardList = (count) => {
    receivedCardList.value = [];
    for (let i = 0; i < count; i++) {
        receivedCardList.value.push({});
    }
};

// 更新receivedCardList中的icon、left和top
const updateReceivedCardList = () => {
    const iconCount = iconList.value.length;
    receivedCardList.value.forEach(card => {
        // 随机选择一个icon
        const randomIconIndex = Math.floor(Math.random() * iconCount);
        card.icon = iconList.value[randomIconIndex].icon;
        card.picIndex = iconList.value[randomIconIndex].picIndex;
        // 生成0~300之间的随机left和top值
        card.left = Math.floor(Math.random() * 301) + 'px';
        card.top = Math.floor(Math.random() * 451) + 'px';
        receivedCardList.value.push({ icon: iconList.value[randomIconIndex].icon, picIndex: iconList.value[randomIconIndex].picIndex, left: Math.floor(Math.random() * 301) + 'px', top: Math.floor(Math.random() * 451) + 'px' });
        receivedCardList.value.push({ icon: iconList.value[randomIconIndex].icon, picIndex: iconList.value[randomIconIndex].picIndex, left: Math.floor(Math.random() * 301) + 'px', top: Math.floor(Math.random() * 451) + 'px' });
    });
};

function selectCard(item, index, imageId) {
    hasOverLayer(imageId, (isOverlayed) => {
        if (isOverlayed) {
            console.log("该元素被遮挡了");
        } else {
            console.log("该元素未被遮挡");
            currentSelectCard.value = item
            allCardList.value.splice(index, 1)
            selectedCard.value.push(item)
            
            // 排序
            selectedCard.value.sort((a, b) => {
                return a.picIndex - b.picIndex
            })
            
            checkRemove()
            
            // 判断胜利或失败条件并弹出提示
            if (selectedCard.value.length > 7) {
                uni.showModal({
                    title: "提示",
                    content: "你输了！",
                    showCancel: false,
                    success: () => {
                        // 重新开始或其他处理逻辑
                    }
                })
            }
            
            if (allCardList.value.length === 0) {
                uni.showModal({
                    title: "提示",
                    content: "你赢了！",
                    showCancel: false,
                    success: () => {
                        // 重新开始或其他处理逻辑
                    }
                })
            }
        }
    });
}


// 判断消除
function checkRemove() {
    let sum = 0
    selectedCard.value.forEach(item => {
        if (item.icon === currentSelectCard.value.icon) {
            sum++
        }
        if (sum > 2) {
            selectedCard.value = selectedCard.value.filter(item => {
                return item.icon != currentSelectCard.value.icon
            })
        }
    })
}

function hasOverLayer(elementId, callback) {
    const query = uni.createSelectorQuery();

    query.select(`#${elementId}`).boundingClientRect((rect) => {
        if (!rect) {
            callback(false); // 如果目标元素未找到，直接返回 false
            return;
        }

        const { left, top, width, height } = rect;
        const points = [
            { x: left + 1, y: top + 1 },
            { x: left + width - 1, y: top + 1 },
            { x: left + 1, y: top + height - 1 },
            { x: left + width - 1, y: top + height - 1 },
        ];

        let hasOverlay = false;
        let checkedPoints = 0;

        points.forEach(({ x, y }) => {
            query.selectAll('*').boundingClientRect((allRects) => {
                checkedPoints++;
                if (allRects.some((el) => el.id !== elementId && x >= el.left && x <= el.right && y >= el.top && y <= el.bottom)) {
                    hasOverlay = true;
                }
                if (checkedPoints === points.length) {
                    callback(hasOverlay);
                }
            }).exec();
        });
    }).exec();
}


onMounted(() => {
    let index = 0
    // 初始化receivedCardList并更新数据
    initializeReceivedCardList(96); // 假设我们需要96 * 3个元素
    updateReceivedCardList();
    receivedCardList.value.forEach(item => {
        allCardList.value.push({ id: index++, icon: item.icon, picIndex: item.picIndex, left: item.left, top: item.top })
    })
})


</script>

<style scoped>
.top-box {
    width: 100%;
    height: 500px;
    background-color: rgb(195, 254, 139);
}
.bottom-box {
    width: 100%;
    height: 200px;
    background-color: rgb(195, 254, 139);
}
.card-box {
    width: 350px;
    height: 100%;
    background-color: rgb(195, 254, 139);
    margin: 0 auto;
    position: relative;
}

.card {
    width: 48px;
    height: 48px;
    display: flex;
    justify-content: center;  /* 水平居中 */
    align-items: center;      /* 垂直居中 */
    line-height: 48px;
    border: 1px solid #666;
    text-align: center;
    position: absolute;
    box-shadow: 0px 0px 6px 0px #545454;
    background-color: #fff;
}

.selected-card {
    width: 48px;
    height: 48px;
    display: flex;
    justify-content: center;  /* 水平居中 */
    align-items: center;      /* 垂直居中 */
    line-height: 48px;
    border: 1px solid #666;
    text-align: center;
    box-shadow: 0px 0px 6px 0px #545454;
    background-color: #fff;
}

.bottom-selected-box {
    width: 350px;
    height: 50px;
    background-color: rgb(150, 91, 27);
    display: flex;
    margin: 0 auto;
}
</style>