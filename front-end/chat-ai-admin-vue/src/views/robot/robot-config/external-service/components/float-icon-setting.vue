<style lang="less" scoped>
.float-icon-setting {
  .form-box {
    .form-item {
      display: flex;
      margin-bottom: 24px;
    }
    .form-item-label {
      width: 120px;
      line-height: 32px;
      margin-right: 4px;
      text-align: right;
    }
    .number-input {
      display: flex;
      align-items: center;
      .input {
        width: 120px;
      }
      .unit {
        font-size: 16px;
        margin-left: 8px;
      }
    }
    .display-types {
      display: flex;
      .display-type {
        position: relative;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        width: 100px;
        height: 100px;
        margin-right: 8px;
        font-size: 14px;
        border-radius: 2px 2px 2px 2px;
        border: 1px solid rgba(0, 0, 0, 0.06);
        background-color: #fff;
        overflow: hidden;
        cursor: pointer;
        &:hover,
        &.is-checked {
          border: 1px solid #2475fc;
        }
        &.is-checked::after {
          content: '';
          position: absolute;
          bottom: -1px;
          right: -1px;
          width: 18px;
          height: 18px;
          background: url('@/assets/svg/check-arrow-filled.svg') no-repeat center;
          border-radius: 0 0 0 2px;
        }

        .type-icon {
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          width: 56px;
          height: 56px;
          img {
            width: auto;
            height: auto;
            max-width: 100%;
            max-height: 100%;
          }
        }
        .type-name {
          color: rgba(0, 0, 0, 0.65);
          margin-top: 4px;
        }
        .checked-icon {
          position: absolute;
          bottom: 0;
          right: 0;
        }
      }
    }
  }
}
</style>

<template>
  <CardBox title="悬浮图标设置">
    <template #icon>
      <MessageOutlined name="quick-Instruction" style="font-size: 16px; color: #262626" />
    </template>
    <div class="float-icon-setting">
      {{ props.title }}
      <div class="form-box">
        <div class="form-item">
          <div class="form-item-label">展示形式：</div>
          <div class="form-content">
            <div class="display-types">
              <div
                class="display-type"
                :class="{ 'is-checked': formData.displayType == 1 }"
                @click="handleDisplayType(1)"
              >
                <div class="type-icon">
                  <img src="@/assets/img/robot/display_type_1.svg" alt="" />
                </div>
                <div class="type-name">简洁</div>
              </div>
              <div
                class="display-type"
                :class="{ 'is-checked': formData.displayType == 2 }"
                @click="handleDisplayType(2)"
              >
                <div class="type-icon">
                  <img src="@/assets/img/robot/display_type_2.svg" alt="" />
                </div>
                <div class="type-name">加文案</div>
              </div>

              <div
                class="display-type"
                :class="{ 'is-checked': formData.displayType == 3 }"
                @click="handleDisplayType(3)"
                v-if="formData.buttonIcon"
              >
                <div class="type-icon">
                  <img
                    :src="formData.buttonIcon"
                    alt=""
                    title=""
                  />
                </div>
                <div class="type-name">自定义</div>
              </div>

              <div
                class="display-type"
                v-if="uploadBoxShow"
              >
                <cu-upload @change="uploadButtonIcon">
                  <div class="type-icon">
                    <PlusOutlined style="font-size: 20px"  />
                  </div>
                </cu-upload>

                <div class="type-name">{{formData.buttonIcon ? '上传图片' : '自定义'}}</div>
              </div>
            </div>
          </div>
        </div>
        <div class="form-item" v-if="formData.displayType == 2">
          <div class="form-item-label">按钮文案：</div>
          <div class="form-content">
            <a-input
              placeholder="请输入按钮文案"
              :maxlength="15"
              style="width: 315px"
              v-model:value="formData.buttonText"
            />
          </div>
        </div>
        <div class="form-item">
          <div class="form-item-label">底边距：</div>
          <div class="form-content">
            <div class="number-input">
              <a-input-number class="input" v-model:value="formData.bottomMargin" />
              <span class="unit">PX</span>
            </div>
          </div>
        </div>
        <div class="form-item">
          <div class="form-item-label">右边距：</div>
          <div class="form-content">
            <div class="number-input">
              <a-input-number class="input" v-model:value="formData.rightMargin" />
              <span class="unit">PX</span>
            </div>
          </div>
        </div>
        <div class="form-item">
          <div class="form-item-label">显示未读消息数：</div>
          <div class="form-content">
            <a-switch
              v-model:checked="formData.showUnreadCount"
              :unCheckedValue="0"
              :checkedValue="1"
            />
          </div>
        </div>
        <div class="form-item">
          <div class="form-item-label">显示新消息提示：</div>
          <div class="form-content">
            <a-switch
              v-model:checked="formData.showNewMessageTip"
              :unCheckedValue="0"
              :checkedValue="1"
            />
          </div>
        </div>
        <div class="form-item">
          <div class="form-item-label"></div>
          <div class="form-content">
            <a-button type="primary" :loading="props.saveLoading" @click="handleSubmit"
              >保存</a-button
            >
          </div>
        </div>
      </div>
    </div>
  </CardBox>
</template>

<script setup>
import { uploadFile } from '@/api/app'
import { reactive, defineProps, toRaw, watch, computed } from 'vue'
import { MessageOutlined, PlusOutlined } from '@ant-design/icons-vue'
import CardBox from './card-box.vue'
import CuUpload from '@/components/cu-upload/cu-upload.vue'

// 定义组件属性
const props = defineProps({
  saveLoading: {
    type: Boolean,
    default: false
  },
  form: {
    type: Object,
    default: () => {}
  }
})

// 定义组件事件
const emit = defineEmits(['save'])

// 响应式数据
const formData = reactive({
  displayType: 1,
  buttonText: '快来聊聊吧~',
  buttonIcon: '',
  bottomMargin: 32,
  rightMargin: 32,
  showUnreadCount: 1,
  showNewMessageTip: 1
})

const handleDisplayType = (type) => {
  formData.displayType = type
}

const uploadBoxShow = computed(() => {
  if(!formData.buttonIcon){
    return true
  }else{
    return formData.displayType == 3
  }
})

const uploadButtonIcon = (files) => {
  let file = files[0]

  handleDisplayType(3)

  uploadFile({
    file: file,
    category: 'icon'
  }).then((res) => {
    formData.buttonIcon = res.data.link
  })
}
// 方法
const handleSubmit = () => {
  console.log(formData.rightMargin)
  if (formData.rightMargin == '' || formData.rightMargin == null) {
    formData.rightMargin = 0
  }

  if (formData.bottomMargin == '' || formData.rightMargin == null) {
    formData.bottomMargin = 0
  }

  emit('save', toRaw(formData))
}

watch(
  () => props.form,
  (newVal) => {
    Object.assign(formData, newVal.floatBtn)
  },
  {
    immediate: true
  }
)

defineExpose({
  handleSubmit
})
</script>
