<template>
  <h1>上传您的简历，由AI帮您推荐CS留学项目</h1>
  <el-upload
      class="upload-demo"
      :on-preview="handlePreview"
      :on-remove="handleRemove"
      :file-list="fileList"
      :before-upload="handleBeforeUpload"
      accept="application/pdf"
  >
    <el-button size="small" type="primary">点击上传简历</el-button>
  </el-upload>

  <el-dialog title="请等待" :visible="loading" :close-on-click-modal="false" :show-close="false">
    <div style="text-align: center">
      <el-spinner :size="loaderSize" :text="loaderText"></el-spinner>
    </div>
  </el-dialog>

  <div>


  </div>

  <el-row>
    <el-col :span="12">
      <el-space direction="vertical">
      <el-card class="box-card" style="width: 800px">
        <template #header>
          <div class="card-header">
            <h2>推荐项目</h2>
          </div>
          <div v-for="school in recommend_project" :key="school.name">
            <h2 class="project-name">{{ school.name }}</h2>
            <div class="project-section">
              <h3 class="section-title">项目介绍</h3>
              <ul class="item-list">
                <li v-for="(value, key) in school.项目介绍" :key="key">
                  <span class="item-key">{{ key }}:</span>
                  <span class="item-value">{{ value }}</span>
                </li>
              </ul>
            </div>
            <div class="project-section">
              <h3 class="section-title">注意事项</h3>
              <div class="sub-notes">
                <h4 class="subsection-title">网申注意事项</h4>
                <ul class="item-list">
                  <li v-for="item in school.注意事项.网申注意事项" :key="item">{{ item }}</li>
                </ul>
              </div>
              <div class="sub-notes">
                <h4 class="subsection-title">其他注意事项</h4>
                <ul class="item-list">
                  <li v-for="item in school.注意事项.其他注意事项" :key="item">{{ item }}</li>
                </ul>
              </div>
            </div>
            <div class="project-section">
              <h3 class="section-title">项目优劣势</h3>
              <div class="sub-pros-cons">
                <h4 class="subsection-title">Pros:</h4>
                <ul class="item-list">
                  <li v-for="item in school.项目优劣势.Pros" :key="item">{{ item }}</li>
                </ul>
              </div>
              <div class="sub-pros-cons">
                <h4 class="subsection-title">Cons:</h4>
                <ul class="item-list">
                  <li v-for="item in school.项目优劣势.Cons" :key="item">{{ item }}</li>
                </ul>
              </div>
            </div>
          </div>


        </template>
        <h2>推荐理由</h2>
        <div v-for="item in recommend_reson" :key="item">{{ item }}</div>
      </el-card>
      </el-space>
    </el-col>
    <!--    ————————————注意这里，上面代码是左侧分栏部分，下面代码是右侧分栏部分——————————-->
    <el-col :span="11" :offset="1">


      <el-card class="box-card" style="width: 800px">
        <h2>您的简历</h2>
        <embed v-if="selectedResume" :src="selectedResume" type="application/pdf" width="80%" height="500px">
        <!-- 添加<embed>元素来显示选中的简历文件 -->
      </el-card>

      <el-card v-for="(value, key) in question" :key="key" class="box-card" style="width: 800px">

        <template #header>
          <div class="card-header">
            <h2>{{ key }}</h2>
          </div>
          <li v-for="items in value" :key="items"> {{items}}</li>
        </template>
        <!-- 添加<embed>元素来显示选中的简历文件 -->
      </el-card>
    </el-col>
  </el-row>

</template>

<script>
export default {
  data() {
    return {
      loading: false, // 加载状态
      loadingText: '请等待...', // 加载状态文本
      result: [], // 存储后端接口返回的数据
      recommend_project: [],
      recommend_reson: [],
      question:[],
      selectedResume: '', // 存储简历的预览内容
    };
  },
  computed: {
    loaderSize() {
      return this.loading ? 'big' : 'medium';
    },
    loaderText() {
      return this.loading ? '请稍等...' : '';
    },
  },
  methods: {
    handleRemove(file, fileList) {
      this.selectedResume = URL.createObjectURL(file.raw); // 存储选中的简历文件的URL
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleBeforeUpload(file) {
      const formData = new FormData();
      formData.append('file', file);

      this.loading = true; // 显示加载状态

      // 发送POST请求到后端接口
      fetch('http://127.0.0.1:8000/global/', {
        method: 'POST',
        body: formData,
      })
          .then(response => response.json())
          .then(data => {
            let transformedData = [];

// Iterate over the keys of the responseData object
            for (let key in data.推荐院校) {
              let item = data.推荐院校[key];
              item.name = key; // Add the key as a 'name' property in each item
              transformedData.push(item);
            }
            this.recommend_project = transformedData;
            this.recommend_reson = data.推荐理由;
            this.loading = false; // 隐藏加载状态

          })
          .catch(error => {
            console.error('上传失败:', error);
            this.loading = false; // 隐藏加载状态
          });

      fetch('http://127.0.0.1:8000/questions/', {
        method: 'POST',
        body: formData,
      })
          .then(response => response.json())
          .then(data => {
            // 处理从后端接收到的数据
            this.question = data;
          })
          .catch(error => {
            console.error('请求失败:', error);
            this.loading = false; // 隐藏加载状态
          });

      // 确保不执行默认的上传操作
      return false;
    },
  },
};
</script>

<style scoped>
.project-container {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.school-container {
  margin-bottom: 20px;
  padding: 10px;
  background-color: #f5f5f5;
  max-width: 800px;
  width: 100%;
}

.project-name {
  margin-bottom: 10px;
  font-size: 24px;
  font-weight: bold;
  text-align: center;
}

.project-section {
  margin-bottom: 20px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.section-title {
  margin-bottom: 10px;
  font-size: 20px;
}

.subsection-title {
  margin-bottom: 5px;
  font-size:18px;
}

.item-list {
  margin-bottom: 1em;
  margin-top: 1em;
  list-style-type: disc;
  padding: 0px;
  margin-left: 1em;
}

.item-list li {
  margin-bottom: 0.5em;
  margin-left: 1.25em;
}

.item-key {
  font-weight: bold;
}

.item-value {
  margin-left: 5px;
  white-space: pre-wrap; /* 处理文本换行 */
  word-break: break-word; /* 处理长单词换行 */
  text-align: left; /* 保持内容左对齐 */
}

/* Element UI样式 */
.upload-demo {
  margin-bottom: 20px;
}

.el-upload__tip {
  margin-top: 10px;
  color: #666;
}

.el-button {
  margin-bottom: 10px;
}

.el-button--primary {
  background-color: #409eff;
  border-color: #409eff;
}

.el-button--primary:hover {
  background-color: #66b1ff;
  border-color: #66b1ff;
}

.el-button--small {
  padding: 6px 12px;
  font-size: 12px;
}
</style>
