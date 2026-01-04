<script>
import { useHeynoteStore } from "@/src/stores/heynote-store"
import { useSettingsStore } from "@/src/stores/settings-store"
import BufferTree from "./BufferTree.vue"
import ResizeHandle from "./ResizeHandle.vue"

export default {
    components: {
        BufferTree,
        ResizeHandle,
    },

    setup() {
        return {
            heynoteStore: useHeynoteStore(),
            settingsStore: useSettingsStore(),
        }
    },

    data() {
        return {
            folderStates: {},  // Track which folders are open
        }
    },

    computed: {
        sidebarWidth() {
            return this.settingsStore.sidebarWidth
        },

        currentBufferPath() {
            return this.heynoteStore.currentBufferPath
        },

        bufferTree() {
            const tree = {
                name: "",
                path: "",
                type: "folder",
                open: true,
                children: []
            }

            const pathSep = window.heynote.buffer.pathSeparator
            const bufferPaths = Object.keys(this.heynoteStore.buffers)

            // Build tree from buffer paths
            for (const bufferPath of bufferPaths) {
                const parts = bufferPath.split(pathSep)
                let currentLevel = tree

                // Navigate/create folder structure
                for (let i = 0; i < parts.length - 1; i++) {
                    const folderName = parts[i]
                    let folder = currentLevel.children.find(
                        c => c.type === "folder" && c.name === folderName
                    )

                    if (!folder) {
                        const folderPath = parts.slice(0, i + 1).join(pathSep)
                        folder = {
                            name: folderName,
                            path: folderPath,
                            type: "folder",
                            open: this.getFolderOpenState(folderPath),
                            children: []
                        }
                        currentLevel.children.push(folder)
                    }
                    currentLevel = folder
                }

                // Add buffer
                const bufferName = parts[parts.length - 1]
                const metadata = this.heynoteStore.buffers[bufferPath]
                currentLevel.children.push({
                    name: metadata?.name || bufferName,
                    path: bufferPath,
                    type: "buffer",
                })
            }

            // Sort: folders first, then alphabetically
            const sortTree = (node) => {
                if (node.children) {
                    node.children.sort((a, b) => {
                        if (a.type !== b.type) {
                            return a.type === "folder" ? -1 : 1
                        }
                        return a.name.localeCompare(b.name)
                    })
                    node.children.forEach(sortTree)
                }
            }
            sortTree(tree)

            return tree
        },
    },

    methods: {
        getFolderOpenState(path) {
            // Default to true (all folders open) if not set
            return this.folderStates[path] !== false
        },

        onBufferSelect(path) {
            this.heynoteStore.openBuffer(path)
        },

        toggleFolder(path) {
            this.folderStates[path] = !this.getFolderOpenState(path)
        },

        onResize(newWidth) {
            this.settingsStore.setSidebarWidth(newWidth)
        },

        createBuffer() {
            this.heynoteStore.showCreateBuffer = true
        },
    },
}
</script>

<template>
    <aside class="sidebar" :style="{ width: sidebarWidth + 'px' }">
        <div class="sidebar-header">
            <h2>Buffers</h2>
            <button
                @click="createBuffer"
                class="create-button"
                title="Create new buffer"
            >+</button>
        </div>
        <div class="sidebar-content">
            <BufferTree
                :treeData="bufferTree"
                :currentPath="currentBufferPath"
                @selectBuffer="onBufferSelect"
                @toggleFolder="toggleFolder"
            />
        </div>
        <ResizeHandle @resize="onResize" />
    </aside>
</template>

<style lang="sass" scoped>
@use "@/src/css/include" as *

.sidebar
    display: flex
    flex-direction: column
    background: var(--panel-background)
    border-right: 1px solid #d4d4d4
    height: 100%
    position: relative
    min-width: 150px
    max-width: 500px

    +dark-mode
        border-right: 1px solid #2a2a2a

    .sidebar-header
        padding: 8px 12px
        border-bottom: 1px solid #d4d4d4
        display: flex
        justify-content: space-between
        align-items: center
        flex-shrink: 0

        +dark-mode
            border-bottom: 1px solid #2a2a2a

        h2
            font-size: 13px
            font-weight: 600
            margin: 0
            color: #444

            +dark-mode
                color: rgba(255, 255, 255, 0.87)

        .create-button
            background: none
            border: none
            font-size: 18px
            cursor: pointer
            padding: 0 6px
            border-radius: 3px
            color: #444
            line-height: 1

            +dark-mode
                color: rgba(255, 255, 255, 0.87)

            &:hover
                background: rgba(0, 0, 0, 0.05)
                +dark-mode
                    background: rgba(255, 255, 255, 0.05)

    .sidebar-content
        flex: 1
        overflow-y: auto
        overflow-x: hidden
</style>
