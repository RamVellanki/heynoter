<script>
import TreeItem from "./TreeItem.vue"

export default {
    components: {
        TreeItem,
    },

    props: {
        treeData: Object,
        currentPath: String,
    },

    data() {
        return {
            selected: 0,
        }
    },

    computed: {
        flattenedTree() {
            const items = []

            const flatten = (node, level) => {
                if (!node) return

                // Add the current node
                items.push({
                    name: node.name,
                    path: node.path,
                    level: level,
                    type: node.type,
                    open: node.open,
                })

                // Add children if folder is open
                if (node.type === "folder" && node.open && node.children) {
                    for (const child of node.children) {
                        flatten(child, level + 1)
                    }
                }
            }

            // Start from level -1 for root so children are at level 0
            if (this.treeData && this.treeData.children) {
                for (const child of this.treeData.children) {
                    flatten(child, 0)
                }
            }

            return items
        },
    },

    methods: {
        onItemClick(item, index) {
            this.selected = index

            if (item.type === "folder") {
                this.$emit('toggleFolder', item.path)
            } else if (item.type === "buffer") {
                this.$emit('selectBuffer', item.path)
            }
        },

        isCurrentBuffer(item) {
            return item.type === "buffer" && item.path === this.currentPath
        },
    },
}
</script>

<template>
    <div class="buffer-tree">
        <TreeItem
            v-for="(item, index) in flattenedTree"
            :key="item.path"
            :name="item.name"
            :path="item.path"
            :level="item.level"
            :type="item.type"
            :selected="selected === index"
            :open="item.open"
            :isCurrent="isCurrentBuffer(item)"
            @click="onItemClick(item, index)"
        />
        <div v-if="flattenedTree.length === 0" class="empty-state">
            No buffers found
        </div>
    </div>
</template>

<style lang="sass" scoped>
@use "@/src/css/include" as *

.buffer-tree
    padding: 4px 0

    .empty-state
        padding: 20px
        text-align: center
        color: #999
        font-size: 13px
        +dark-mode
            color: rgba(255, 255, 255, 0.5)
</style>
