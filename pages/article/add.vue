<template>
  <view class="page_article_add">
    <uni-forms :modelValue="form">
      <uni-forms-item label="标题" name="title">
        <uni-easyinput v-model="form.title" placeholder="请输入标题" />
      </uni-forms-item>

      <uni-forms-item label="分类" name="category">
        <uni-data-select
          v-model="form.category"
          :localdata="typeOptions"
          placeholder="请选择分类"
        />
      </uni-forms-item>

      <uni-forms-item label="封面图" name="cover_image">
        <view class="diy_field diy_img" v-if="form.cover_image">
          <image :src="form.cover_image" @click="change_cover_image" style="width: 100px; height: 100px;" />
        </view>
        <view class="diy_field diy_img" v-else>
          <view class="btn_add_img" @click="change_cover_image">
            <text>+</text>
          </view>
        </view>
      </uni-forms-item>

      <uni-forms-item label="摘要" name="summary">
        <uni-easyinput v-model="form.summary" placeholder="请输入摘要" />
      </uni-forms-item>

      <uni-forms-item label="内容" name="content">
        <uni-easyinput type="textarea" v-model="form.content" placeholder="请输入内容" />
      </uni-forms-item>
    </uni-forms>

    <view class="button-group">
      <button type="primary" @click="submitArticle">保存</button>
      <button class="btn-cancel" @click="cancel">取消</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      form: {
        title: '',
        category: '',
        cover_image: '',
        summary: '',
        content: '',
      },
      typeOptions: [],
      imageFiles: []
    };
  },
  methods: {
    onLoad() {
      this.getArticleTypes();
    },
    getArticleTypes() {
      this.$get('~/api/article_type/get_list', { page: 1, size: 0 }, (res) => {
        console.log("📦 分类接口原始返回：", res);
        if (res?.result?.list?.length > 0) {
          this.typeOptions = res.result.list.map(item => ({
            value: item.name, // 分类名（如“心理”）
            text: item.name
          }));
          console.log("✅ 生成的下拉数据：", this.typeOptions);
        } else {
          this.$toast('暂无分类数据', 'warning');
        }
      });
    },
    change_cover_image() {
      const _self = this;
      uni.chooseImage({
        count: 1,
        sizeType: ['original', 'compressed'],
        sourceType: ['album'],
        success: function(res) {
          const tempFilePath = res.tempFilePaths[0];
          uni.uploadFile({
            url: 'http://127.0.0.1:5000/api/article/upload',
            filePath: tempFilePath,
            name: 'file',
            success: function(uploadRes) {
              const result = JSON.parse(uploadRes.data);
              if (result?.result?.url) {
                _self.form.cover_image = result.result.url;
                _self.imageFiles = [{ url: result.result.url }];
                _self.$toast('封面图上传成功', 'success');
              } else {
                _self.$toast('封面图上传失败: 无URL返回', 'error');
              }
            },
            fail() {
              _self.$toast('封面图上传失败', 'error');
            }
          });
        },
        fail(err) {
          console.error("❌ 选择图片失败：", err);
        }
      });
    },
    submitArticle() {
      console.log("📤 准备提交的表单数据:", this.form);

      if (!this.form.title || !this.form.title.trim()) {
        this.$toast('标题不能为空', 'error');
        return;
      }
      if (!this.form.summary || !this.form.summary.trim()) {
        this.$toast('摘要不能为空', 'error');
        return;
      }
      if (!this.form.content || !this.form.content.trim()) {
        this.$toast('内容不能为空', 'error');
        return;
      }
      if (!this.form.cover_image || !this.form.cover_image.trim()) {
        this.$toast('封面图不能为空', 'error');
        return;
      }

      const submitData = {
        title: this.form.title,
        description: this.form.summary,         // ✅ 摘要（后端识别）
        content: this.form.content,
        type: this.form.category || '',         // ✅ 分类（后端识别）
        cover_image: this.form.cover_image,
        img: this.form.cover_image              // ✅ 兼容旧字段
      };

      console.log("📦 最终提交数据：", submitData);

      this.$post('~/api/article/add?', submitData, (res) => {
        if (res.result) {
          this.$toast('添加成功', 'success');
          uni.navigateBack();
        } else {
          this.$toast('添加失败', 'error');
        }
      });
    },


    cancel() {
      uni.navigateBack();
    }
  }
};
</script>

<style scoped>
.page_article_add {
  padding: 20px;
}
.button-group {
  margin-top: 30px;
  text-align: center;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 20px;
}
.btn-cancel {
  background-color: #f0f0f0;
  color: #333;
}
</style>
