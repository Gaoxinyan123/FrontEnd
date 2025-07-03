<template>
  <view class="page_article" id="article_details">
  <view v-if="user_group === '管理员'" style="padding: 10px; text-align: right;">
    <button type="primary" @click="goEditArticle">编辑文章</button>
  </view>
    <!-- ✅ 顶部封面图（如果有） -->
    <view v-if="obj.img" class="cover-image">
      <image
        :src="$fullUrl(obj.img)"
        mode="aspectFill"
        class="cover-img"
      />
    </view>

    <!-- 文章详情模块(开始) -->
    <template v-if="$check_action('/article/details', 'get')">

      <div_article
        style="background-color: #fff"
        :obj="obj"
        class="mb"
      ></div_article>

      <!-- 推荐文章 -->
      <uni-card title="推荐文章" v-if="$check_action('/comment/list', 'get')">
        <list_article :list="list_article"></list_article>
      </uni-card>

      <!-- 文章评论列表 -->
      <uni-card title="文章点评" v-if="$check_action('/comment/list', 'get')">
        <list_comment
          style="background-color: #fff"
          :list="list_comment"
          :obj="form_comment"
          :current_user="current_user"
          @refresh="get_comment"
        ></list_comment>
      </uni-card>

      <!-- 发表评论 -->
      <view class="pa" v-if="$check_action('/comment/list', 'add')">
        <button
          class="link"
          @click="handle_comment"
        >
          发表评论
        </button>
      </view>

    </template>
    <!-- 文章详情模块(结束) -->
  </view>
</template>

<script>
import bar_title from "@/components/diy/bar_title.vue";
import list_article from "@/components/diy/list_article.vue";
import div_article from "@/components/diy/div_article.vue";
import list_comment from "@/components/diy/list_comment.vue";

import mixin from "@/libs/mixins/page.js";

export default {
  mixins: [mixin],
  components: {
    bar_title,
    list_article,
    div_article,
    list_comment,
  },
  data() {
    return {
      url_get_obj: getApp().globalData.host + "/api/article/get_obj?",
      field: "article_id",
      query: {
        article_id: 0,
      },
      obj: {
        article_id: 0,
        title: "",
        type: "",
        hits: 0,
        create_time: "",
        update_time: "",
        source: "",
        url: "",
        tag: "",
        content: "",
        img: "", // ✅ 用于显示封面图
        description: "",
        praise_len: 0,
      },
      list_article: [],
      list_comment: [],
      form: {
        content: "",
      },
      form_comment: {
        source_table: "article",
        source_field: "article_id",
        source_id: 0,
        reply_to_id: 0,
      },
      current_user: {
        user_id: null,
        user_group: '',
      },
    };
  },
  methods: {
    onEditorReady() {
      const that = this;
      uni
        .createSelectorQuery()
        .select("#editor")
        .context((res) => {
          this.editorCtx = res.context;
          this.editorCtx.setContents({
            html: this.form.content,
            success: (res) => {
              console.log(res);
            },
            fail: (res) => {
              console.log(res);
            },
          });
        })
        .exec();
    },
    // 获取推荐文章（同类型，排除当前文章）
    get_article() {
  const currentType = this.obj.type;
  const currentId = Number(this.obj.article_id); // 确保是数字

  console.log("当前文章ID：", currentId);
  console.log("当前文章类型：", currentType);

  this.$get(
    getApp().globalData.host + "/api/article/get_list?",
    {
      page: 1,
      size: 3,
      type: currentType, // 筛选类型相同
    },
    (json) => {
      if (json.result) {
        const rawList = json.result.list || [];
        const filteredList = rawList.filter(item => Number(item.article_id) !== currentId);

        console.log("原始推荐列表：", rawList);
        console.log("过滤后推荐列表：", filteredList);

        this.list_article = filteredList;
      }
    }
  );
},
    get_comment() {
      const options = getCurrentPages()[getCurrentPages().length - 1].options;
      const query = {
        source_table: "article",
        source_field: "article_id",
        source_id: options.article_id,
        orderby: "create_time desc",
        reply_to_id: "0",
      };
      this.$get(getApp().globalData.host + "/api/comment/get_list?", query, (json) => {
        if (json.result) {
          const list_comment = json.result.list;
          list_comment.forEach(o => o.list_reply = []);
          this.add_reply(list_comment).then(list => {
            this.list_comment = list;
          });
        }
      });
    },
    add_reply(list) {
      return new Promise(resolve => {
        for (let idx = 0; idx < list.length; idx++) {
          const obj = list[idx];
          this.$get(
            getApp().globalData.host + "/api/comment/get_list?",
            {
              source_table: "article",
              source_field: "article_id",
              source_id: obj.article_id,
              orderby: "create_time desc",
              reply_to_id: obj.comment_id,
            },
            (res) => {
              if (res.result) {
                obj.list_reply = res.result.list;
              }
            }
          );
        }
        resolve(list);
      });
    },
   get_obj_after(json) {
        this.add_hits(this.obj);
        let obj = this.obj;
        this.get_comment(obj);
        this.form_comment.source_id = obj.article_id;

        // ✅ 等文章加载完后再加载推荐文章
        this.get_article(); 
      },
    add_hits(obj) {
      this.$post(
        getApp().globalData.host + "/api/article/set?article_id=" + obj.article_id,
        { hits: obj.hits + 1 },
        (res) => {
          obj.hits += 1;
        }
      );
    },
    goEditArticle() {
      if (this.user_group === '管理员') {
        uni.navigateTo({ url: '/pages/article/edit?article_id=' + this.obj.article_id });
      } else {
        this.$toast('只有管理员可以编辑文章', 'error');
      }
    },
    checkLoginBeforeComment(e) {
      if (!this.$store.state.user || !this.$store.state.user.user_id) {
        this.$toast('请登录后再操作');
        e.preventDefault && e.preventDefault();
        e.stopPropagation && e.stopPropagation();
        return false;
      }
    },
    handle_comment() {
      if (!this.$store.state.user || !this.$store.state.user.user_id || this.$store.state.user.user_group === '游客') {
        this.$toast('请登录后再操作');
        return;
      }
      // 跳转到评论编辑页
      uni.navigateTo({
        url: `/pages/comment/edit?source_table=article&source_field=article_id&source_id=${this.obj.article_id}`
      });
    },
  },
  onLoad() {
    // 初始化当前用户信息
    this.current_user = {
      user_id: this.$store.state.user.user_id,
      user_group: this.$store.state.user.user_group
    };
    
    this.get_article();
    this.get_comment();
  },
};
</script>
.edit-button-top {
  position: sticky;
  top: 0;
  z-index: 999;
  background: #fff;
  padding: 10px;
  border-bottom: 1px solid #eee;
  text-align: right;
}

.edit-button-top button {
  background-color: #22B8B8;
  color: white;
  font-size: 16px;
  font-weight: bold;
}

<style>
#article_details .bar_title {
  background: none;
}
#article_details .recommend {
  background-color: #fff;
  overflow: hidden;
}
#article_details .link {
  text-align: center;
  padding: 0.5rem 0;
  border: 1px solid #dbdbdb;
  border-radius: 0.5rem;
}

.cover-image {
  width: 100%;
  height: 200px;
  overflow: hidden;
  margin-bottom: 10px;
}

.cover-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}
</style>
