<template>
    <div class="page-wrapper">
        <div class="page-header">
            <div v-for="(item, index) in someThings" class="thing_item grid-stack-item" :style="{
                width: item.size.width + 'px',
                height: item.size.height + 'px',
            }" :key="index" :gs-w="`${item.size.width}px`" :gs-h="`${item.size.height}px`">
                <div class="grid-stack-item-content">
                    <!-- <span>{{ item.description }}</span> -->
                </div>
            </div>
        </div>

        <div class="page-content">
            <div class="page-content_wrapper">
                <div class="grid-stack" @click="selectItemRotate"></div>
            </div>
        </div>

        <div class="page-footer">
            <div style="display: flex;justify-content: space-around;width: 100%;">
                <button>设置</button>
                <button>确认</button>
                <button @click="clear">清空</button>
                <div class="trash" id="trash">回收站</div>
            </div>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { GridStack } from 'gridstack';
import { onMounted, ref } from "vue";

import { EnumHelper } from './utils/helper'
import { ISomeThing, SomeThingType } from './types'
import { nextTick } from 'vue';
import { onUnmounted } from 'vue';

const someThings = ref<ISomeThing[]>()
const SomeThingTypeHelper = new EnumHelper(SomeThingType)

const grid = ref<GridStack>()

const scale = ref({ x: 1, y: 1 });
// @ts-ignore
const pageMaxWidth = 600;  // 页面最大宽度
const updateScaleCssVariable = () => {
    document.body.style.setProperty('--global-scale-x', `${scale.value.x}`);
    document.body.style.setProperty('--global-scale-y', `${scale.value.y}`);
}
// @ts-ignore
const changeScaleX = (scaleX: number) => {
    scale.value.x = scaleX;
    updateScaleCssVariable();
};
// 页面尺寸变化
const resizeObserver = new ResizeObserver((_entries) => {
    // const { width } = _entries[0].contentRect;
    // if (width < pageMaxWidth) {
    //     changeScaleX(pageMaxWidth / width);
    // }
});

onMounted(() => {
    // 初始化物品数据
    ; (someThings.value ??= []).push(
        ...Array.from({ length: 2 }).map((_, index) => ({
            id: index + 1,
            type: SomeThingType.柜子,
            size: { width: 60, height: 120, depth: 1 },
            description: SomeThingTypeHelper.getKeyByValue(SomeThingType.柜子)! + index,
        })),
        ...Array.from({ length: 5 }).map((_, index) => ({
            id: index + 1,
            type: SomeThingType.板凳,
            size: { width: 40, height: 40, depth: 1 },
            description: SomeThingTypeHelper.getKeyByValue(SomeThingType.板凳)! + index,
        })),
    )


    // 初始化拖拽
    nextTick(() => { // 等待页面渲染完成 | 否则无法获取到header上的元素
        grid.value = GridStack.init({
            minRow: 3,
            acceptWidgets: true,
            cellHeight: 50,
            removable: '#trash', // drag-out delete class
            // disableResize: true,
        });
        grid.value.float(true);
        GridStack.setupDragIn('.page-header .grid-stack-item', {
            appendTo: 'body', helper: (ev) => {
                const node = (ev.target as HTMLElement)?.cloneNode() as HTMLElement

                // 获取原始元素的宽高
                // const { width, height } = (ev.target as HTMLElement).getBoundingClientRect()
                // node.style.width = width + 'px'
                // node.style.height = height + 'px'

                node.appendChild(document.createTextNode('……'))
                return node
            }
        });

        updateScaleCssVariable()
    })

    resizeObserver.observe(document.body);
})




onUnmounted(() => {
    grid.value?.destroy()
    resizeObserver.disconnect();
})

// 选中元素
function selectItemRotate(ev: Event) {
    const selected = ev.target as HTMLElement

    if (selected?.classList.contains('grid-stack-item-content')) {
        // 清除其他选中
        document.querySelectorAll('.grid-stack-item-content').forEach(item => {
            item.classList.remove('selected')
        })
        selected.classList.add('selected')

        grid.value?.getGridItems().forEach(item => {
            if (item === selected?.parentElement) {
                // console.log(item.gridstackNode);
                grid.value?.rotate(item)
            }
        })
    }
}

// 清空
function clear() {
    grid.value?.removeAll()
}

</script>

<style lang="less" scoped>
.page-wrapper {
    height: 100%;
    overflow: hidden;

    display: flex;
    flex-direction: column;
    justify-content: space-between;

    .page-header {
        display: flex;
        width: 100%;
        overflow: auto;
        background-color: #00000068;
        flex-shrink: 0;

        .thing_item {
            display: flex;
            flex-shrink: 0;
            align-items: center;
            justify-content: center;
            margin: 10px;
            font-size: 20px;
            background-color: #fff;
            position: relative;

            .grid-stack-item-content {
                height: 100%;
                width: 100%;
            }
        }
    }


    .page-content {
        --grid-size: 10px;
        --move-item-deviation: 0px;
        flex-grow: 1;
        overflow: auto;

        &_wrapper {
            width: 600px;
            transform: translate(0, 0) scale(var(--global-scale-x), var(--global-scale-y));
            transform-origin: 0 0;
        }

        .grid-stack {
            --temp-size: calc(var(--grid-size) + var(--move-item-deviation) * 2);
            background-image: linear-gradient(to right, transparent calc(var(--temp-size) - 1px), #ccc 100%),
                linear-gradient(to bottom, transparent calc(var(--temp-size) - 1px), #ccc 100%);
            background-repeat: repeat;
            background-size: var(--temp-size) var(--temp-size);
            border-radius: 5px;
        }
    }

    .page-footer {
        padding: 10px;
        display: flex;
        flex-shrink: 0;
        background-color: #fff;
        flex-direction: column;
        gap: 10px;

        .trash {
            background-color: rgb(255, 72, 72);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            padding: 0 40px;
            color: #fff;
            border-radius: 5px;
            user-select: none;
        }
    }
}

:deep(.grid-stack-item-content) {
    background-color: #18bc9c;
    user-select: none;

    &.selected {
        background-color: #f39c12;
    }
}

:deep(.ui-resizable-handle) {
    display: none;
}
</style>