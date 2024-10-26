<template>
    <view class="box">
        <!-- top -->
        <view class="top-box">
            <view class="card-box">
                <view class="card" @click="selectCard(item, index, $event)" v-for="(item, index) in allCardList"
                    :key="index" :style="{ left: item.left, top: item.top }">
                    <img :src="`/src/assets/${item.icon}.svg`" alt="icon" style="width: 20px; height: 20px;" />
                </view>
            </view>
        </view>
        <!-- bottom -->
        <view class="bottom-selected-box">
            <view class="selected-card" v-for="(item, index) in selectedCard" :key="index">
                <img :src="`/src/assets/${item.icon}.svg`" alt="icon" style="width: 20px; height: 20px;" />
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

function selectCard(item, index, event) {
    let isHover = hasOverLayer(event.srcElement)
    if (isHover) {
        return
    }
    currentSelectCard.value = item
    allCardList.value.splice(index, 1)
    selectedCard.value.push(item)
    // 排序
    selectedCard.value.sort((a, b) => {
        return a.picIndex - b.picIndex
    })
    checkRemove()
    if (selectedCard.value.length > 7) {
        alert("你输了！")
    }
    if (allCardList.value.length == 0) {
        alert("你赢了！")
    }
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

// 监测元素是否被覆盖
function hasOverLayer(element) {
    let document = element.ownerDocument,
        rect = element.getBoundingClientRect(), // 获取目标的矩形信息
        x = rect.x,
        y = rect.y,
        width = rect.width,
        height = rect.height;
    x |= 0;
    y |= 0;
    width |= 0;
    height |= 0;
    let elements = [
        document.elementFromPoint(x + 1, y + 1),
        document.elementFromPoint(x + width - 1, y + 1),
        document.elementFromPoint(x + 1, y + height - 1),
        document.elementFromPoint(x + width - 1, y + height - 1)
    ];
    return elements.filter((el) => el != null).some((el) => el !== element && !element.contains(el));
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
    background-color: greenyellow;
}

.card-box {
    width: 350px;
    height: 100%;
    background-color: #dddaaa;
    margin: 0 auto;
    position: relative;
}

.card {
    width: 48px;
    height: 48px;
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
    line-height: 48px;
    border: 1px solid #666;
    text-align: center;
    box-shadow: 0px 0px 6px 0px #545454;
    background-color: #fff;
    display: inline-block;
}

.bottom-selected-box {
    width: 350px;
    height: 50px;
    background-color: antiquewhite;
    margin: 0 auto;
}
</style>