<script setup>
import { ref } from 'vue'
import ChannelSelect from '@/views/article/components/ChannelSelect.vue'
import { Plus } from '@element-plus/icons-vue'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'
import {
  artEditService,
  artGetDetailService,
  artPublishService
} from '@/api/article'
import { baseURL } from '@/utils/request'
import axios from 'axios'

// 控制抽屉显示隐藏
const visibleDrawer = ref(false)

// 默认数据
const defaultForm = {
  title: '', // 标题
  cate_id: '', // 分类id
  cover_img: '', // 封面图片 file 对象
  content: '', // string 内容
  state: '' // 状态
}

// 准备数据
const formModel = ref({
  ...defaultForm
})

// 图片上传相关逻辑
const imgUrl = ref('')
const onSelectFile = (uploadFile) => {
  imgUrl.value = URL.createObjectURL(uploadFile.raw) // 预览图片
  formModel.value.cover_img = uploadFile.raw // 将raw中的图片传入表单
}

// 提交
const emit = defineEmits(['success'])
const onPublish = async (state) => {
  // 将已发布函数草稿状态，存入formModel
  formModel.value.state = state

  // 注意：当前接口，需要的是 formData 对象
  // 将普通对象 => 转换成 => formData 对象
  const fd = new FormData()
  for (let key in formModel.value) {
    fd.append(key, formModel.value[key])
  }

  // 发送请求
  if (formModel.value.id) {
    // 编辑操作
    await artEditService(fd)
    ElMessage.success('修改成功')
    visibleDrawer.value = false
    emit('success', 'edit')
  } else {
    console.log(formModel.value.cover_img)
    // 添加操作
    await artPublishService(fd)
    ElMessage.success('添加成功')
    visibleDrawer.value = false
    // 通知父组件,添加成功了
    emit('success', 'add')
  }
}

// 编辑器Ref
const editorRef = ref()

// 将网络图片地址转换为File对象
async function imageUrlToFile(url, fileName) {
  try {
    // 第一步：使用axios获取网络图片数据
    const response = await axios.get(url, { responseType: 'arraybuffer' })
    const imageData = response.data

    // 第二步：将图片数据转换为Blob对象
    const blob = new Blob([imageData], {
      type: response.headers['content-type']
    })

    // 第三步：创建一个新的File对象
    return new File([blob], fileName, { type: blob.type })
  } catch (error) {
    console.error('将图片转换为File对象时发生错误:', error)
    throw error
  }
}

const open = async (row) => {
  visibleDrawer.value = true // 显示抽屉

  if (row.id) {
    // 需要基于 row.id 发送请求，获取编辑对应的详情数据，进行回显
    const res = await artGetDetailService(row.id)
    formModel.value = res.data.data
    // 图片需要单独回显
    imgUrl.value = baseURL + formModel.value.cover_img
    // 注意：提交给后台，需要的数据格式，是file对象格式
    // 需要将网络图片地址 => 转换成 file 对象，存储起来,将来便于提交
    formModel.value.cover_img = await imageUrlToFile(
      imgUrl.value,
      formModel.value.cover_img
    )
  } else {
    formModel.value = { ...defaultForm } // 基于默认的数据，重置form对象
    // 这里重置了表单的数据，但是图片上传的img地址和富文本编辑器的内容 => 需要手动重置
    imgUrl.value = ''
    editorRef.value.setHTML('')
  }
}

defineExpose({
  open
})
</script>

<template>
  <el-drawer
    v-model="visibleDrawer"
    :title="formModel.id ? '编辑文章' : '添加文章'"
    direction="rtl"
    size="50%"
  >
    <!-- 发表文章表单 -->
    <el-form :model="formModel" ref="formRef" label-width="100px">
      <el-form-item label="文章标题" prop="title">
        <el-input v-model="formModel.title" placeholder="请输入标题"></el-input>
      </el-form-item>
      <el-form-item label="文章分类" prop="cate_id">
        <channel-select
          v-model="formModel.cate_id"
          width="100%"
        ></channel-select>
      </el-form-item>
      <el-form-item label="文章封面" prop="cover_img">
        <el-upload
          class="avatar-uploader"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="onSelectFile"
        >
          <img v-if="imgUrl" :src="imgUrl" class="avatar" alt="" />
          <el-icon v-else class="avatar-uploader-icon">
            <Plus />
          </el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item label="文章内容" prop="content">
        <div class="editor">
          <quill-editor
            ref="editorRef"
            v-model:content="formModel.content"
            content-type="html"
            theme="snow"
          ></quill-editor>
        </div>
      </el-form-item>
      <el-form-item>
        <el-button @click="onPublish('已发布')" type="primary">发布</el-button>
        <el-button @click="onPublish('草稿')" type="info">草稿</el-button>
      </el-form-item>
    </el-form>
  </el-drawer>
</template>
<style scoped lang="scss">
.editor {
  width: 100%;
  :deep(.ql-editor) {
    min-height: 200px;
  }
}
.avatar-uploader {
  :deep() {
    .avatar {
      width: 178px;
      height: 178px;
      display: block;
    }
    .el-upload {
      border: 1px dashed var(--el-border-color);
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;
      transition: var(--el-transition-duration-fast);
    }
    .el-upload:hover {
      border-color: var(--el-color-primary);
    }
    .el-icon.avatar-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 178px;
      height: 178px;
      text-align: center;
    }
  }
}
</style>
