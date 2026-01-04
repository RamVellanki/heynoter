<script>
export default {
    props: {
        name: String,
        path: String,
        level: Number,
        type: String,  // "folder" or "buffer"
        selected: Boolean,
        open: Boolean,
        isCurrent: Boolean,
    },

    watch: {
        selected() {
            if (this.selected) {
                // scrollIntoViewIfNeeded is not supported in all browsers
                if (this.$el.scrollIntoViewIfNeeded) {
                    this.$el.scrollIntoViewIfNeeded({
                        behavior: "auto",
                        block: "nearest",
                    })
                } else {
                    this.$el.scrollIntoView({
                        behavior: "auto",
                        block: "nearest",
                    })
                }
            }
        }
    },

    computed: {
        className() {
            const classes = {
                'tree-item': true,
                selected: this.selected,
            }

            if (this.type === 'folder') {
                classes.folder = true
                classes.open = this.open
            } else if (this.type === 'buffer') {
                classes.buffer = true
                classes.current = this.isCurrent
            }

            return classes
        },

        style() {
            return {
                "--indent-level": this.level,
            }
        }
    },
}
</script>

<template>
    <div
        :class="className"
        :style="style"
        @click="$emit('click')"
    >
        <span class="name">{{ name }}</span>
    </div>
</template>

<style lang="sass" scoped>
@use "@/src/css/include" as *

.tree-item
    padding: 3px 6px
    font-size: 13px
    padding-left: calc(18px + var(--indent-level) * 16px)
    display: flex
    align-items: center
    cursor: pointer
    scroll-margin-top: 5px
    scroll-margin-bottom: 5px
    user-select: none

    // Base styles
    color: #444
    +dark-mode
        color: rgba(255, 255, 255, 0.87)

    &:hover
        background-color: #f1f1f1
        +dark-mode
            background-color: #39393a

    // Folder-specific styles
    &.folder
        background-size: 13px
        background-repeat: no-repeat
        background-position-y: 5px
        background-position-x: calc(2px + var(--indent-level) * 16px)
        background-image: url('@/assets/icons/caret-right.svg')

        +dark-mode
            background-image: url('@/assets/icons/caret-right-white.svg')

        &.open
            background-image: url('@/assets/icons/caret-down.svg')
            +dark-mode
                background-image: url('@/assets/icons/caret-down-white.svg')

        &.selected
            background-color: #48b57e
            color: #fff
            background-image: url('@/assets/icons/caret-right-white.svg')

            &.open
                background-image: url('@/assets/icons/caret-down-white.svg')

            &:hover
                background-color: #40a773
                +dark-mode
                    background-color: #1f6f47

    // Buffer-specific styles
    &.buffer
        &.current
            background-color: #48b57e
            color: #fff
            font-weight: 600
            +dark-mode
                background-color: #1b6540

            &:hover
                background-color: #40a773
                +dark-mode
                    background-color: #1f6f47

    .name
        flex-grow: 1
        overflow: hidden
        text-overflow: ellipsis
        white-space: nowrap
</style>
