<template>
    <div style="flex:1">
        <hotState></hotState>
    </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
    import hotState from "./hotState.vue";
    module.exports = {
        data () {
            return {}
        },
        components: {
          hotState
        }
    }
</script>