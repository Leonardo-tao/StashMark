<template>
    <a-layout class="container">
        <!-- 书签区域 -->
        <a-layout-content class="bookmarks-area">
                <draggable v-model="unclassifiedBookmarks" group="bookmarks" item-key="id" class="draggable-grid"
                    :component-data="{ tag: 'div' }">
                    <template #item="{ element }">
                            <div class="bookmark-item" @mouseenter="showPreview(element, $event)" @mouseleave="hidePreview">
                                <a-avatar :src="element.icon" shape="square" :size="48" class="bookmark-icon">
                                    <template v-if="!element.icon" #icon>
                                        <LinkOutlined />
                                    </template>
                                </a-avatar>
                                <span class="bookmark-title">{{ element.title }}</span>
                            </div>
                    </template>
                </draggable>

            <!-- 书签预览卡片 -->
            <div v-if="previewVisible" class="preview-card" :style="previewStyle">
                <a-avatar :src="currentPreview?.icon" shape="square" :size="64">
                    <template v-if="!currentPreview?.icon" #icon>
                        <LinkOutlined />
                    </template>
                </a-avatar>
                <h4 class="preview-title">{{ currentPreview?.title }}</h4>
                <a-typography-text type="secondary" class="preview-url">
                    {{ currentPreview?.url }}
                </a-typography-text>
            </div>
        </a-layout-content>

        <!-- 文件夹区域 -->
        <a-layout-footer class="folder-area">
            <draggable v-model="folders" group="folders" item-key="id" class="folder-container">
                <template #item="{ element }">
                    <div class="folder-item" @contextmenu.prevent="(e) => e.preventDefault()">
                        <a-dropdown :trigger="['contextmenu']" :getPopupContainer="() => $el">
                            <div class="folder-header">
                                <span class="folder-name">{{ element.name }}</span>
                                <a-badge :count="element.bookmarks.length" class="badge-count" />
                            </div>
                            <template #overlay>
                                <a-menu>
                                    <a-menu-item @click="handleRenameFolder(element)">
                                        <template #icon>
                                            <EditOutlined />
                                        </template>
                                        重命名
                                    </a-menu-item>
                                    <a-menu-item @click="handleDeleteFolder(element.id)" danger>
                                        <template #icon>
                                            <DeleteOutlined />
                                        </template>
                                        删除
                                    </a-menu-item>
                                </a-menu>
                            </template>
                        </a-dropdown>

                        <draggable v-model="element.bookmarks" group="bookmarks" class="folder-content" item-key="id">
                            <template #item="{ element: bookmark }">
                                <a-avatar :src="bookmark.icon" shape="square" :size="24" class="folder-bookmark">
                                    <template v-if="!bookmark.icon" #icon>
                                        <LinkOutlined />
                                    </template>
                                </a-avatar>
                            </template>
                        </draggable>
                    </div>
                </template>
            </draggable>
        </a-layout-footer>

        <!-- 重命名模态框 -->
        <a-modal v-model:visible="renameModalVisible" title="重命名文件夹" @ok="confirmRename">
            <a-input v-model:value="newFolderName" placeholder="请输入新名称" />
        </a-modal>
    </a-layout>
</template>

<script setup>
import { ref, reactive } from 'vue';
import draggable from 'vuedraggable';
import {
    LinkOutlined,
    EditOutlined,
    DeleteOutlined
} from '@ant-design/icons-vue';
import { Modal } from 'ant-design-vue';

// 书签数据
const unclassifiedBookmarks = ref([
    { id: 1, title: 'GitHub', url: 'https://github.com', icon: '/logo.svg' },
    { id: 2, title: 'Google', url: 'https://google.com', icon: '/logo.svg' },
    { id: 3, title: 'Baidu', url: 'https://baidu.com', icon: '/logo.svg' },
    { id: 4, title: 'Bing', url: 'https://bing.com', icon: '/logo.svg' },
    { id: 5, title: 'YouTube', url: 'https://youtube.com', icon: '/logo.svg' },
    { id: 6, title: 'Twitter', url: 'https://twitter.com', icon: '/logo.svg' },
    { id: 7, title: 'Facebook', url: 'https://facebook.com', icon: '/logo.svg' },
    { id: 8, title: 'Instagram', url: 'https://instagram.com', icon: '/logo.svg' },
    { id: 9, title: 'LinkedIn', url: 'https://linkedin.com', icon: '/logo.svg' },
]);
// 文件夹数据
const folders = ref([
    {
        id: 1,
        name: '工作',
        bookmarks: [
            { id: 3, title: 'Notion', url: 'https://notion.so' }
        ]
    },
    {
        id: 2,
        name: '学习',
        bookmarks: []
    },
]);

// 预览相关状态
const previewVisible = ref(false);
const currentPreview = ref(null);
const previewStyle = reactive({
    left: '0px',
    top: '0px'
});

// 文件夹操作相关状态
const renameModalVisible = ref(false);
const newFolderName = ref('');
let currentFolder = null;

const showPreview = (bookmark, event) => {
    currentPreview.value = bookmark;
    previewVisible.value = true;
    updatePreviewPosition(event);
};

const hidePreview = () => {
    previewVisible.value = false;
};

const updatePreviewPosition = (event) => {
    previewStyle.left = `${event.clientX + 15}px`;
    previewStyle.top = `${event.clientY + 15}px`;
};

const handleRenameFolder = (folder) => {
    currentFolder = folder;
    newFolderName.value = folder.name;
    renameModalVisible.value = true;
};

const confirmRename = () => {
    if (currentFolder && newFolderName.value) {
        currentFolder.name = newFolderName.value;
        renameModalVisible.value = false;
    }
};

const handleDeleteFolder = (folderId) => {
    Modal.confirm({
        title: '确认删除文件夹?',
        content: '该操作会同时删除文件夹内的所有书签',
        okText: '确认',
        cancelText: '取消',
        onOk: () => {
            folders.value = folders.value.filter(f => f.id !== folderId);
        }
    });
};
</script>

<style scoped>

* {
    box-sizing: border-box;
}
.container {
    height: calc(100vh - 80px);
}

.bookmarks-area {
    height: 50%;
    padding: 24px;
    background: #fff;
    overflow: auto;
}

/* 移除或注释掉原来的grid样式 */
/*
.draggable-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 16px;
}
*/

/* 可以添加以下样式优化布局 */
.bookmarks-grid {
    width: 100%;
}

.bookmark-item {
    display: inline-block;
    flex-direction: column;
    align-items: center;
    padding: 8px;
    cursor: move;
    transition: all 0.3s;
}

.bookmark-item:hover {
    background: #f5f5f5;
    border-radius: 8px;
}

.bookmark-title {
    margin-top: 8px;
    font-size: 12px;
    text-align: center;
    word-break: break-all;
}

.folder-area {
    height: 50%;
    padding: 16px;
    background: #f0f2f5;
    border-top: 1px solid #e8e8e8;
}

.folder-container {
    display: flex;
    gap: 16px;
    height: 100%;
    overflow-x: auto;
}

.folder-item {
    flex-shrink: 0;
    width: 180px;
    height: 160px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.folder-header {
    padding: 12px;
    border-bottom: 1px solid #f0f0f0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: context-menu;
}

.folder-name {
    font-weight: 500;
}

.folder-content {
    padding: 12px;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 8px;
    height: calc(160px - 45px);
    overflow: hidden;
}

.preview-card {
    position: fixed;
    z-index: 1000;
    padding: 16px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
    width: 240px;
    text-align: center;
}

.preview-title {
    margin: 12px 0 4px;
    font-size: 14px;
}

.preview-url {
    display: block;
    font-size: 12px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
</style>