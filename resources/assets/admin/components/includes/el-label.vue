<template>
    <span>
        {{ $t(label) }}
        <el-tooltip v-if="helpText" popper-class="sidebar-popper" effect="dark" placement="top">
            <div slot="content" v-html="$t(helpText)"></div>
            <i class="tooltip-icon" :class="iconClass"></i>
        </el-tooltip>
    </span>
</template>

<script>
export default {
    name: 'el-label-slot',
    props: ['label', 'helpText', 'icon'],
    computed: {
        iconClass() {
            return `el-icon-${this.icon || 'info'}`;
        }
    },
}
</script>

<style>
    .sidebar-popper {
        max-width: 300px;
        line-height: 1.5;
    }
</style>