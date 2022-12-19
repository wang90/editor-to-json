<template>
    <font source="user">{{ value }}</font>
</template>
<script setup lang="ts">
import { defineProps } from 'vue'
const props = defineProps({
    value: String,
})
</script> 

<style lang="less" scoped>
font {
    &[source="user"] {
      color: #000;
    }
    &[source="model"] {
      color: red;
    }
}
</style>