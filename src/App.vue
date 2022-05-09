<template>
  <div id="app">
    <div class="container">
      <el-card>
        <div
          slot="header"
          class="clearfix"
        >
          <el-row type="flex">
            <el-upload
              class="upload-demo"
              action=""
              :auto-upload="false"
              :on-change="handleChange"
              :show-file-list="false"
            >
              <el-button
                size="small"
                type="primary"
              >选择图片</el-button>
            </el-upload>
            <el-button
              style="margin-left: 16px;"
              type="primary"
              :disabled="src === ''"
              size="small"
              @click="handleDeal"
            >灰度处理</el-button>
          </el-row>
        </div>
        <el-row>
          <el-col :span="12">
            <el-image
              ref="imageRef"
              :src="src"
              lazy
            >
              <div
                slot="error"
                class="image-slot"
              >
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
          </el-col>
        </el-row>
        <el-row
          type="type"
          :gutter="16"
        >
          <el-col :span="12">
            <el-image
              ref="imageRef"
              :src="src2"
              lazy
            >
              <div
                slot="error"
                class="image-slot"
              >
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
          </el-col>
          <el-col :span="12">
            <el-image
              :class="{ 'el-image--gray': isGray}"
              :src="src3"
              lazy
            >
              <div
                slot="error"
                class="image-slot"
              >
                <i class="el-icon-picture-outline"></i>
              </div>
            </el-image>
          </el-col>
        </el-row>
      </el-card>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      src: '',
      src2: '',
      src3: '',
      isGray: false
    }
  },
  methods: {
    handleChange(e) {
      this.isGray = false
      const file = e.raw
      let url = ''
      if (window.createObjectURL != undefined) {
        url = window.createObjectURL(file);
      } else if (window.URL != undefined) {
        url = window.URL.createObjectURL(file);
      } else if (window.webkitURL != undefined) {
        url = window.webkitURL.createObjectURL(file);
      }
      this.src = this.src2 = this.src3 = url
    },
    // 灰度处理
    handleDeal() {
      // 第一种处理方式，通过样式处理
      this.isGray = true

      // 第二种处理方式，通过canvas处理
      this.gray()
    },
    gray() {
      const ob = this.$refs.imageRef
      const img = this.$refs.imageRef.$el.children[0]
      // 获取图片的高度和宽度
      const height = ob.imageHeight
      const width = ob.imageWidth

      // 创建一个canvas节点
      const canvas = document.createElement('canvas')
      if (canvas.getContext) {
        const context = canvas.getContext('2d')
        canvas.width = width
        canvas.height = height
        // 绘制图片
        context.drawImage(img, 0, 0)

        // 获取图片的rgb数据
        const c = context.getImageData(0, 0, width, height)
        // 进行 rgb数据的处理
        for (var i = 0; i < c.height; i++) {
          for (var j = 0; j < c.width; j++) {
            var x = (i * 4) * c.width + (j * 4);
            var r = c.data[x];
            var g = c.data[x + 1];
            var b = c.data[x + 2];
            var gr = r * 0.299 + g * 0.578 + b * 0.114
            c.data[x] = gr
            c.data[x + 1] = gr
            c.data[x + 2] = gr // (r + g + b) / 3;
          }
        }
        // 将重新赋值的像素点重新归位到context中
        context.putImageData(c, 0, 0, 0, 0, c.width, c.height);
        // 生成src路径
        this.src2 = canvas.toDataURL()
        // 删除canvas节点
        document.removeChild(canvas)
      }
    }
  }
}
</script>

<style>
.container {
  margin-top: 200px;
  height: 666px;
  width: 1200px;
  margin: 0 auto;
}

.el-image img {
  transition: all cubic-bezier(0.86, 0, 0.07, 1) 0.8s;
}

.el-image--gray > img {
  filter: grayscale(100%);
}
</style>
