<script>
export default {
    data() {
        return {
            isResizing: false,
            startX: 0,
            startWidth: 0,
        }
    },

    methods: {
        onMouseDown(e) {
            this.isResizing = true
            this.startX = e.clientX
            this.startWidth = parseInt(getComputedStyle(this.$el.parentElement).width)

            document.addEventListener('mousemove', this.onMouseMove)
            document.addEventListener('mouseup', this.onMouseUp)
            document.body.style.cursor = 'col-resize'
            document.body.style.userSelect = 'none'
            e.preventDefault()
        },

        onMouseMove(e) {
            if (!this.isResizing) return

            const delta = e.clientX - this.startX
            const newWidth = this.startWidth + delta

            // Constrain width between 150px and 500px
            const constrainedWidth = Math.min(Math.max(newWidth, 150), 500)
            this.$emit('resize', constrainedWidth)
        },

        onMouseUp() {
            this.isResizing = false
            document.removeEventListener('mousemove', this.onMouseMove)
            document.removeEventListener('mouseup', this.onMouseUp)
            document.body.style.cursor = ''
            document.body.style.userSelect = ''
        },
    },

    beforeUnmount() {
        // Cleanup
        document.removeEventListener('mousemove', this.onMouseMove)
        document.removeEventListener('mouseup', this.onMouseUp)
        document.body.style.cursor = ''
        document.body.style.userSelect = ''
    },
}
</script>

<template>
    <div
        class="resize-handle"
        @mousedown="onMouseDown"
    ></div>
</template>

<style lang="sass" scoped>
@use "@/src/css/include" as *

.resize-handle
    position: absolute
    right: 0
    top: 0
    bottom: 0
    width: 4px
    cursor: col-resize
    background: transparent
    transition: background 150ms
    z-index: 10

    &:hover
        background: #48b57e
        +dark-mode
            background: rgba(255, 255, 255, 0.1)
</style>
