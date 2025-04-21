<template>
  <a-layout class="s-container">
    <a-row class="bookmarks" :gutter="[16, 24]" justify="space-evenly">
      <a-col
        v-for="item in unclassifiedBookmarks"
        :key="item.id"
        draggable="true"
        @dragstart="handleDragStart($event, item)"
      >
        <a-card hoverable :body-style="{ padding: '12px' }">
          <template #cover>
            <img
              alt="website preview"
              :src="item.icon"
              style="height: 120px; object-fit: contain"
            />
          </template>
          <a-card-meta :title="item.title">
            <template #description>{{ item.url }}</template>
          </a-card-meta>
        </a-card>
      </a-col>
    </a-row>
    <a-row class="folders" :gutter="[16, 16]">
      <a-col
        v-for="folder in folders"
        :key="folder.id"
        :xs="24"
        :sm="12"
        :md="8"
        :lg="6"
      >
        <a-card
          hoverable
          @dragover.prevent
          @drop="handleDrop(folder)"
          :body-style="{ padding: '12px', position: 'relative' }"
          @contextmenu.prevent="handleFolderRightClick($event, folder)"
          class="folder-card"
        >
				<!-- <a-badge 
            :count="folder.bookmarks.length" 
            :number-style="{ backgroundColor: '#1890ff' }"
            style="position: absolute; right: 8px; top: 8px; height: 20px;width: 20px;;"
          /> -->
          <folder-outlined style="font-size: 24px" />
          <h3>{{ folder.name }}</h3>
          
        </a-card>

        <!-- 右键菜单 -->
        <div
          v-if="contextMenu.visible"
          :style="{
            position: 'fixed',
            left: contextMenu.x + 'px',
            top: contextMenu.y + 'px',
            zIndex: 1000,
            background: 'white', // 添加背景
            boxShadow: '0 2px 8px rgba(0,0,0,0.15)', // 添加阴影
          }"
        >
          <a-menu>
            <a-menu-item @click="handleRename">重命名</a-menu-item>
            <a-menu-item @click="handleDelete">删除</a-menu-item>
            <a-menu-item @click="handleDisband">解散文件夹</a-menu-item>
          </a-menu>
        </div>
      </a-col>
      <a-col :xs="24" :sm="12" :md="8" :lg="6">
        <a-card
          hoverable
          :body-style="{
            padding: '12px',
            cursor: 'pointer',
            textAlign: 'center',
          }"
          @click="handleCreate"
          class="new-folder-card"
        >
          <plus-outlined style="font-size: 32px; color: #1890ff" />
          <h3>新建文件夹</h3>
        </a-card>
      </a-col>
    </a-row>
    <a-modal
      v-model:visible="renameModal.visible"
      title="重命名文件夹"
      @ok="confirmRename"
    >
      <a-input v-model:value="renameModal.newName" placeholder="请输入新名称" />
    </a-modal>
    <a-modal
      v-model:visible="createModal.visible"
      title="新建文件夹"
      @ok="formRef.validate().then(confirmCreate)"
    >
      <a-form ref="formRef" :model="createModal">
        <a-form-item 
          name="newName"
          :rules="[
            { required: true, message: '请输入文件夹名称' },
            { validator: checkFolderUnique }
          ]"
        >
          <a-input 
            v-model:value="createModal.newName" 
            placeholder="请输入文件夹名称" 
          />
        </a-form-item>
      </a-form>
    </a-modal>
  </a-layout>
</template>

<script setup lang="ts">
import { Modal } from "ant-design-vue";
import { ref, reactive, onMounted, onUnmounted } from "vue";
import { withBase } from "vitepress";

const icon = withBase("/logo.svg");

// 书签数据
const unclassifiedBookmarks = ref([
  { id: 1, title: "GitHub", url: "https://github.com", icon },
  { id: 2, title: "Google", url: "https://google.com", icon },
  { id: 3, title: "Baidu", url: "https://baidu.com", icon },
  { id: 4, title: "Bing", url: "https://bing.com", icon },
  { id: 5, title: "YouTube", url: "https://youtube.com", icon },
  { id: 6, title: "Twitter", url: "https://twitter.com", icon },
  { id: 7, title: "Facebook", url: "https://facebook.com", icon },
  { id: 8, title: "Instagram", url: "https://instagram.com", icon },
  { id: 9, title: "LinkedIn", url: "https://linkedin.com", icon },
]);
// 文件夹数据
// 新增图标引入
import { FolderOutlined, PlusOutlined } from "@ant-design/icons-vue";
import { create } from "domain";

// 右键菜单状态
const contextMenu = reactive({
  visible: false,
  x: 0,
  y: 0,
  currentFolder: null,
});

// 右键点击处理
const handleFolderRightClick = (event, folder) => {
  contextMenu.visible = true;
  contextMenu.x = event.clientX;
  contextMenu.y = event.clientY;
  contextMenu.currentFolder = folder;
};

// 重命名模态框状态
const renameModal = reactive({
  visible: false,
  newName: "",
});

const createModal = reactive({
  visible: false,
  newName: "",
});

// 菜单操作处理函数
const handleRename = () => {
  renameModal.newName = contextMenu.currentFolder.name;
  renameModal.visible = true;
};

const confirmRename = () => {
  contextMenu.currentFolder.name = renameModal.newName;
  renameModal.visible = false;
};

const handleCreate = () => {
  createModal.visible = true;
};

// 新增表单引用
const formRef = ref();

// 验证规则
const checkFolderUnique = (_, value) => {
  const isExist = folders.value.some(
    f => f.name.trim().toLowerCase() === value.trim().toLowerCase()
  );
  return isExist ? Promise.reject('已存在同名文件夹') : Promise.resolve();
};

// 简化后的提交逻辑
const confirmCreate = () => {
  folders.value.push({
    id: Date.now(),
    name: createModal.newName,
    bookmarks: []
  });
  createModal.newName = "";
  createModal.visible = false;
  formRef.value.resetFields();
};

const handleDelete = () => {
  Modal.confirm({
    title: "确认删除",
    content: "确定要永久删除这个文件夹吗？",
    onOk: () => {
      const index = folders.value.findIndex(
        (f) => f.id === contextMenu.currentFolder.id
      );
      if (index > -1) {
        folders.value.splice(index, 1);
      }
    },
  });
};

const handleDisband = () => {
  Modal.confirm({
    title: "解散文件夹",
    content: "确定要解散这个文件夹吗？书签将移回未分类区域",
    onOk: () => {
      unclassifiedBookmarks.value.push(...contextMenu.currentFolder.bookmarks);
      folders.value = folders.value.filter(
        (f) => f.id !== contextMenu.currentFolder.id
      );
    },
  });
};

// 更新文件夹数据（添加测试数据）
const folders = ref([
  {
    id: 1,
    name: "工作",
    bookmarks: [{ id: 3, title: "Notion", url: "https://notion.so", icon }],
  },
  {
    id: 2,
    name: "学习",
    bookmarks: [],
  },
  // 新增测试文件夹
  {
    id: 3,
    name: "娱乐",
    bookmarks: [
      { id: 10, title: "YouTube", url: "https://youtube.com", icon },
      { id: 11, title: "Netflix", url: "https://netflix.com", icon },
    ],
  },
]);

// 新增拖放相关逻辑
let draggedBookmark = ref(null);

// 书签拖动处理
const handleDragStart = (event, bookmark) => {
  draggedBookmark.value = bookmark;
};

// 文件夹放置处理
const handleDrop = (folder) => {
  if (draggedBookmark.value) {
    // 添加到文件夹
    folder.bookmarks.push({ ...draggedBookmark.value });
    // 从未分类中移除
    const index = unclassifiedBookmarks.value.findIndex(
      (b) => b.id === draggedBookmark.value.id
    );
    if (index > -1) {
      unclassifiedBookmarks.value.splice(index, 1);
    }
  }
};

// 修复右键菜单问题
const closeContextMenu = () => {
  contextMenu.visible = false;
};

onMounted(() => {
  document.addEventListener("click", closeContextMenu);
});

onUnmounted(() => {
  document.removeEventListener("click", closeContextMenu);
});
</script>

<style scoped>
/* 添加拖拽时卡片透明效果 */
.a-card[draggable="true"] {
  cursor: grab;
  transition: opacity 0.2s;
}

.a-card[draggable="true"]:active {
  cursor: grabbing;
  opacity: 0.6;
}

/* 添加拖放效果 */
.folder-card[dragover] {
  box-shadow: 0 0 8px rgba(24, 144, 255, 0.6);
  transform: scale(1.02);
}

/* 书签卡片缩小为0.5倍并改为正方形 */
.a-card[draggable="true"] {
  width: 90px !important;
  height: 90px !important;
}

/* 书签卡片内容调整 */
.a-card[draggable="true"] .ant-card-body {
  padding: 8px !important;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

/* 书签封面图缩小 */
.a-card[draggable="true"] img {
  height: 60px !important;
  width: 60px !important;
}

/* 隐藏书签文字 */
.a-card[draggable="true"] .ant-card-meta {
  display: none;
}

/* 文件夹卡片调整为正方形 */
.folder-card {
  width: 220px !important;
  height: 220px !important;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

/* 文件夹图标调整 */
.folder-card .anticon {
  font-size: 32px !important;
  margin-bottom: 8px;
}

/* 文件夹标题调整 */
.folder-card h3 {
  font-size: 14px;
  margin: 0;
}

/* 调整栅格响应式布局 */
.a-col {
  --lg-span: 4;  /* 原为6，变窄1/3 */
  --xl-span: 3;   /* 原为6，变窄1/2 */
}

@media (min-width: 992px) {
  .a-col.lg-6 {
    flex: 0 0 calc(100% / 24 * var(--lg-span));
    max-width: calc(100% / 24 * var(--lg-span));
  }
}

@media (min-width: 1200px) {
  .a-col.xl-3 {
    flex: 0 0 calc(100% / 24 * var(--xl-span));
    max-width: calc(100% / 24 * var(--xl-span));
  }
}


/* 右键菜单圆角修复 */
div[style*="position: fixed"] {
  border-radius: 6px !important;
  overflow: hidden;
}

/* 新建文件夹弹窗修正 */
.ant-modal-body {
  padding: 24px;
}
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  overflow: hidden;
}

.s-container {
  height: calc(100vh - 64px);
}

.bookmarks,
.folders {
  height: 50%;
  padding: 16px;
}

.bookmarks {
  overflow-y: auto;
  background-color: #f0f2f5;
}

.folders {
  background-color: #fff;
}

/* 新增文件夹卡片样式 */
.folder-card {
  text-align: center;
  transition: all 0.3s;
}

.count {
  position: absolute;
  right: 12px;
  bottom: 12px;
  color: #888;
  font-size: 12px;
}

/* 右键菜单圆角 */
.context-menu {
  border-radius: 6px;
}

/* 菜单项悬停效果 */
.ant-menu-item:hover {
  background: #f5f5f5 !important;
  border-radius: 4px;
}

/* 新建文件夹卡片样式 */
.new-folder-card {
  border: 2px dashed #d9d9d9;
  transition: all 0.3s;
}

.new-folder-card:hover {
  border-color: #1890ff;
}
</style>
