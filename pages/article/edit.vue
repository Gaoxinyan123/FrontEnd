<template>
  <view class="page_article_edit">
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
      <button v-if="isEdit" type="warn" class="btn-delete" @click="deleteArticle">删除文章</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      form: {
        article_id: null,
        title: '',
        category: '',
        cover_image: '',
        summary: '',
        content: '',
      },
      imageFiles: [],
      uploading: false,
      isEdit: false,
      typeOptions: [],
    };
  },
  onLoad(options) {
    if (options && options.article_id) {
      this.isEdit = true;
      this.$get('~/api/article/get_obj?', { article_id: options.article_id }, (res) => {
      const obj = res.result;
      if (obj && obj.title) {
        this.form.article_id = obj.article_id;
        this.form.title = obj.title;
        this.form.category = obj.type || '';
        console.log("📌 文章加载成功，分类为：", obj.type);
        this.form.cover_image = obj.img || '';
        this.form.summary = obj.description;
        this.form.content = obj.content;
        this.imageFiles = obj.img ? [{ url: obj.img }] : [];
      } else {
        console.log("⚠️ 未找到对应文章对象");
      }
    });

    }
    this.getArticleTypes();
  },
  methods: {
    getArticleTypes() {
      this.$get('~/api/article_type/get_list', { page: 1, size: 0 }, (res) => {
        if (res?.result?.list?.length > 0) {
          this.typeOptions = res.result.list.map(item => ({
            value: item.name,
            text: item.name
          }));
          console.log("📌 分类选项 typeOptions 加载成功：", this.typeOptions);
        } else {
          this.$toast('暂无分类数据', 'warning');
          console.log("⚠️ 分类数据为空");
        }
      });
    },

    deleteArticle() {
      if (!this.form.article_id) {
        this.$toast('未获取到文章ID，无法删除', 'error');
        return;
      }
      uni.showModal({
        title: '确认删除',
        content: '确定要删除这篇文章吗？',
        success: (res) => {
          if (res.confirm) {
            this.$get(`http://127.0.0.1:5000/api/article/del?article_id=${this.form.article_id}`, {}, (json) => {
                    if (json.result) {
                    this.$toast('删除成功', 'success');
                    uni.reLaunch({
                      url: '/pages/article/index'  // ✅ 这里改为跳转首页
                    });
                  } else {
                this.$toast(json.msg || '删除失败', 'error');
              }
            });
          }
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
              if (result && result.result && result.result.url) {
                _self.form.cover_image = result.result.url;
                _self.imageFiles = [{ url: result.result.url }];
                _self.$toast('封面图上传成功', 'success');
              } else {
                _self.$toast('封面图上传失败: 无URL返回', 'error');
              }
            },
            fail: function(err) {
              _self.$toast('封面图上传失败', 'error');
            }
          });
        },
        fail: function(err) {
          console.error("❌ 选择图片失败：", err);
        }
      });
    },
    submitArticle() {
      if (this.uploading) {
        this.$toast('图片上传中，请稍候再保存', 'warning');
        return;
      }
      if (this.imageFiles.length > 0) {
        const file = this.imageFiles[0];
        this.form.cover_image = file.url || file.path || '';
      }
      if (!this.form.title || !this.form.content) {
        this.$toast('标题和内容不能为空', 'error');
        return;
      }
      const submitData = {
        ...this.form,
        img: this.form.cover_image,
        type: this.form.category,
        description: this.form.summary,
      };

      console.log("📤 正在提交的文章数据：", submitData);  // ✅ 打印提交内容
      console.log("📌 当前分类：", this.form.category);

      if (this.isEdit) {
        this.$post(`~/api/article/set?article_id=${this.form.article_id}`, submitData, (res) => {
          console.log("📝 修改文章结果：", res);
          if (res.result) {
            this.$toast('修改成功', 'success');
            uni.navigateBack();
          }
        });
      } else {
        this.$post('~/api/article/add?', submitData, (res) => {
          console.log("🆕 添加文章结果：", res);
          if (res.result) {
            this.$toast('添加成功', 'success');
            uni.navigateBack();
          }
        });
      }
    },

    cancel() {
      uni.navigateBack();
    },
  },
};
</script>

<style scoped>
.page_article_edit {
  padding: 20px;
}
.button-group {
  margin-top: 30px;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 15px;
}

.button-group button {
  min-width: 120px;
  font-size: 15px;
  padding: 10px 15px;
  border-radius: 6px;
}

.btn-cancel {
  background-color: #f1f1f1;
  color: #333;
  border: 1px solid #ccc;
}

.btn-delete {
  background-color: #e74c3c;
  color: white;
  border: none;
}

.btn-delete:hover {
  background-color: #c0392b;
}

</style>