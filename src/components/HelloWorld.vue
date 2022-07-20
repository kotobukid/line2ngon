<template lang="pug">
    .hello
        h1 {{ msg }}
</template>

<script lang="ts">
import {Component, Prop, Vue} from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
    @Prop() private msg!: string;
}
</script>

<style scoped lang="less">
</style>
