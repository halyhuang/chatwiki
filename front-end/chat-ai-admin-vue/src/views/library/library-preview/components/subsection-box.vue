<template>
  <div class="subsection-box">
    <div class="subsection-box-title">
      分段预览
      <span>共{{ props.total }}个分段</span>
    </div>
    <div
      class="list-item"
      v-for="(item,index) in props.paragraphLists"
      :key="item.id"
      @click="handleToTargetPage(item, index)"
    >
      <div class="top-block">
        <div class="title">
          <!-- id：{{ item.id }} -->
          分段{{ index + 1 }}
          <div class="title-block">
            {{ item.title }}
          </div>
          <span>共{{ item.word_total }}个字符</span>
          <span>
            嵌入状态：{{ item.status_text }}<LoadingOutlined v-if="item.status == 3" />
            <a-tooltip v-if="item.status == 2 && item.errmsg" :title="item.errmsg">
              <strong class="cfb363f">原因<ExclamationCircleOutlined class="err-icon cfb363f"/></strong>
            </a-tooltip>
          </span>
          <span>
            知识图谱状态：{{ item.graph_status_text }}
            <a-tooltip v-if="item.graph_status == 3 && item.graph_err_msg" :title="item.graph_err_msg">
              <strong class="cfb363f">原因<ExclamationCircleOutlined class="err-icon cfb363f"/></strong>
            </a-tooltip>
          </span>
        </div>
        <div class="right-opration">
          <a-tooltip>
            <template #title>重新转换</template>
            <SyncOutlined @click="toReSegmentationPage(item, index)" />
          </a-tooltip>
          <a-dropdown placement="bottomRight">
            <MoreOutlined />
            <template #overlay>
              <a-menu>
                <a-menu-item>
                  <div @click.stop="handleOpenEditModal(item)">编辑</div>
                </a-menu-item>
                <a-menu-item>
                  <div @click.stop="hanldleDelete(item.id)">删除</div>
                </a-menu-item>
              </a-menu>
            </template>
          </a-dropdown>
        </div>
      </div>
      <div class="content-box" v-if="item.question">Q：{{ item.question }}</div>
      <div class="content-box" v-if="item.answer">A：{{ item.answer }}</div>
      <div class="content-box" v-html="item.content"></div>
      <div class="fragment-img" v-viewer>
        <img v-for="(item, index) in item.images" :key="index" :src="item" alt="" />
      </div>
    </div>
  </div>
</template>
<script setup>
import { reactive, ref, computed, createVNode } from 'vue'
import { message } from 'ant-design-vue'
import { ExclamationCircleOutlined, MoreOutlined, SyncOutlined, LoadingOutlined } from '@ant-design/icons-vue'
import { Modal } from 'ant-design-vue'
import { deleteParagraph, editParagraph } from '@/api/library'

const emit = defineEmits([
  'handleDelParagraph',
  'handleScrollTargetPage',
  'openEditSubscription',
  'handleConvert'
])
const props = defineProps({
  paragraphLists: {
    type: Array,
    default: () => []
  },
  total: {
    type: [Number, String],
    default: 0
  }
})

const toReSegmentationPage = (item, index) => {
  let { id, title, content, question, answer, images } = item
  Modal.confirm({
    title: '重新转换确认',
    icon: null,
    content: `确定要重新转换【分段${index + 1}】吗?`,
    onOk() {
      return new Promise((resolve, reject) => {
        editParagraph({ id, title, content, question, answer, images })
          .then((res) => {
            emit('handleConvert', item)
            resolve()
          })
          .catch(() => {
            resolve()
          })
      })
    },
    onCancel() {}
  })
}

const handleOpenEditModal = (item) => {
  emit('openEditSubscription', item)
}
const hanldleDelete = (id) => {
  Modal.confirm({
    title: '提示',
    icon: createVNode(ExclamationCircleOutlined),
    content: '确认是否删除该分段?',
    onOk() {
      return new Promise((resolve, reject) => {
        deleteParagraph({ id })
          .then((res) => {
            message.success('删除成功')
            emit('handleDelParagraph', id)
            resolve()
          })
          .catch(() => {
            resolve()
          })
      })
    },
    onCancel() {}
  })
}

const handleToTargetPage = (item, index) => {
  emit('handleScrollTargetPage', {
    page_num: item.page_num,
    index
  })
}
defineExpose({ handleOpenEditModal })
</script>
<style lang="less" scoped>
.subsection-box {
  background: #f2f4f7;
  padding: 14px 16px;
  width: 100%;
  .subsection-box-title {
    display: flex;
    align-items: center;
    font-size: 14px;
    line-height: 22px;
    font-weight: 600;
    color: #242933;
    span {
      color: #7a8699;
      font-weight: 400;
      margin-left: 8px;
    }
  }
  .list-item {
    margin-top: 8px;
    width: 100%;
    background: #fff;
    border-radius: 2px;
    padding: 16px;
    .top-block {
      display: flex;
      align-items: center;
      justify-content: space-between;
      .title {
        display: flex;
        align-items: center;
        font-size: 14px;
        line-height: 22px;
        font-weight: 600;
        color: #000000;
        width: fit-content;
        span {
          color: #8c8c8c;
          font-weight: 400;
          margin-left: 8px;
        }
        .title-block {
          max-width: 320px;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          margin-left: 4px;
        }
      }
      .right-opration {
        display: flex;
        align-items: center;
        gap: 14px;
        line-height: 22px;
        .v-line {
          display: block;
          width: 1px;
          height: 12px;
          background: #d8dde6;
          margin: 0 16px;
        }
      }
    }
    .content-box {
      color: #595959;
      font-size: 14px;
      font-weight: 400;
      line-height: 22px;
      margin-top: 8px;
      white-space: pre-wrap;
      word-wrap: break-word;
    }
    .fragment-img {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-top: 8px;
      img {
        width: 80px;
        height: 80px;
        border-radius: 6px;
        cursor: pointer;
      }
    }
  }
}
@keyframes flash-border {
  0%,
  100% {
    background: transparent;
  }
  50% {
    background: #c8d9f4;
  }
}

.flash-border {
  background: #c8d9f4;
  animation: flash-border 1s infinite; /* 持续时间1秒，无限次重复 */
}
.cfb363f {
  color: #fb363f !important;
}
.err-icon {
  margin-left: 4px !important;
}
</style>
