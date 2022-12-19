<template>
    <span :data-nodeid="nodeid">
        <base-font  v-for="( f ) in values" 
            :from="f.from"
            :value="f.value"
        ></base-font>
    </span>
</template>
<script setup lang="ts">
import BaseFont from './BaseFont.vue';
import { defineProps } from 'vue';
const props = defineProps({
    nodeid: String,
    values: Array,
})
</script> 

<style lang="less" scoped>

</style>