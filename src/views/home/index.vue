<script setup lang="ts">
import HelloWorld from '@/components/HelloWorld.vue'
</script>

<template>
    <div style="height: 100%;display: flex;flex-direction: column;align-items: center;padding: 30px 0 80px 0;">
        <div style="display: flex;flex-wrap: wrap;justify-content: center;margin-bottom: auto;">
            <a href="https://vitejs.dev" target="_blank">
                <img src="/vite.svg" class="logo" alt="Vite logo" />
            </a>
            <a href="https://vuejs.org/" target="_blank">
                <img src="@/assets/vue.svg" class="logo vue" alt="Vue logo" />
            </a>
        </div>
        <HelloWorld msg="Vite + Vue" />
    </div>
</template>

<style scoped lang="less">
.logo {
    height: 10em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;

    &:hover {
        filter: drop-shadow(0 0 2em #646cffaa);
    }

    &.vue:hover {
        filter: drop-shadow(0 0 2em #42b883aa);
    }
}
</style>
