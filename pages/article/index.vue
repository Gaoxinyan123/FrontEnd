<template>
  <view class="page_article" id="article_list">
    <template v-if="$check_action('/article/list', 'get')">
      <!-- 新增文章按钮（仅管理员可见） -->
      <view v-if="current_user && current_user.user_group === '管理员'" class="float-add-btn">
        <button type="primary" @click="goAddArticle">➕ 添加</button>
      </view>
      <!-- 搜索栏 -->
      <uni-search-bar
        placeholder="搜索文章"
        @confirm="search_"
        @cancel="cancel"
        cancelText="取消"
        @input="input($event, 'title')"
      >
        <uni-icons slot="searchIcon" color="#999999" size="18" type="home" />
      </uni-search-bar>

      <!-- 分类选择 -->
      <view style="padding: 10px;">
        <uni-data-select
          @change="searchType"
          v-model="query.type"
          :localdata="types"
        ></uni-data-select>
      </view>

      <!-- 排序栏 -->
      <view class="list_orderby_wrap">
        <view class="list_orderby">
          <bar_orderby
            v-for="(o, i) in list_orderby"
            :key="i"
            :text="o.name"
            :direction="o.direction"
            @update:direction="val => updateDirection(i, val)"
            @handle="handleOrderby"
          ></bar_orderby>
        </view>
      </view>

      <!-- 文章列表 -->
      <list_article
        style="background-color: #fff"
        :list="list"
        :current_user="current_user"
        show-delete
        @delete="get_list"
        class="mb"
      ></list_article>

      <!-- 分页器 -->
      <uni-pagination
        style="padding: 10px"
        title="分页器"
        show-icon="true"
        :total="count"
        :pageSize="query.size"
        :current="query.page"
        @change="page_change"
      ></uni-pagination>
    </template>
  </view>
</template>

<script>
import list_article from "@/components/diy/list_article.vue";
import bar_orderby from "@/components/diy/bar_orderby.vue";
import list_tab from "@/components/diy/list_tab.vue";
import mixin from "@/libs/mixins/page.js";

export default {
  mixins: [mixin],
  components: {
    list_article,
    bar_orderby,
    list_tab,
  },
  data() {
    return {
      url_get_list: "~/api/article/get_list?like=0",
      list: [],
      query: {
        title: "",
        page: 1,
        size: 4,
        type: "",
        orderby: ""
      },
      list_orderby: [
        {
          name: "发布时间",
          direction: "",
          command_asc: "`create_time` asc",
          command_desc: "`create_time` desc",
        },
        {
          name: "点赞数",
          direction: "",
          command_asc: "`praise_len` asc",
          command_desc: "`praise_len` desc",
        },
        {
          name: "浏览量",
          direction: "",
          command_asc: "`hits` asc",
          command_desc: "`hits` desc",
        },
      ],
      types: [
        {
          value: "",
          text: "全部"
        }
      ],
      current_user: getApp().globalData.current_user || {},
    };
  },
  methods: {
    
    updateDirection(i, val) {
      this.list_orderby[i].direction = val;
    },
    get_article_type() {
      this.$get(
        "~/api/article_type/get_list",
        {
          page: 1,
          size: 0,
        },
        (res) => {
          if (res.result) {
            let list = res.result.list;
            list.map((obj) => {
              this.types.push({
                value: obj.name,
                text: obj.name
              });
            });
          }
        }
      );
    },
    input(e, key) {
      this.query[key] = e.value;
    },
    search_() {
      this.query.page = 1;
      this.get_list();
    },
    searchType(v) {
      this.query.type = v;
      this.query.page = 1;
      this.get_list();
    },
    cancel() {
      this.query.title = "";
      this.search_();
    },
    handleOrderby(o) {
      this.list_orderby.map((val) => {
        if (val.name !== o.text) val.direction = "";
      });
      const obj = this.list_orderby.find((val) => val.name === o.text);
      if (o.direction === "") {
        this.query.orderby = "";
      } else if (o.direction === "up") {
        this.query.orderby = obj.command_desc;
      } else if (o.direction === "down") {
        this.query.orderby = obj.command_asc;
      }
      console.log("排序字段:", this.query.orderby);
      this.search_();
    },
    goAddArticle() {
      uni.navigateTo({ url: '/pages/article/add' }); // 👈 指向新增文章页
    },
    get_list() {
      console.log("当前用户:", this.current_user);
      let query = { ...this.query };
      Object.keys(query).forEach((k) => {
        if (!query[k]) delete query[k];
      });
      this.$get(this.url_get_list, query, (json) => {
        if (!json?.result?.list) {
          this.list = [];
          this.count = 0;
        } else {
          this.list = json.result.list;
          this.count = json.result.count;
        }
      });
    },
  },
  mounted() {
    console.log("当前用户：", this.current_user);
console.log("用户组：", this.current_user.user_group);
    this.current_user = getApp().globalData.current_user || {};
      this.get_article_type();
      this.get_list();
  },
};
</script>

<style scoped>
#article_list {}
.pager {
  margin-top: 1rem;
}
.list_orderby_wrap {
  padding: 0 10px;
  margin-bottom: 10px;
}
.list_orderby {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}
.list_orderby .bar_orderby {
  padding: 0.5rem 1rem;
  background-color: #22b8b8;
  color: #fff;
  border-radius: 0.5rem;
  cursor: pointer;
  display: flex;
  align-items: center;
}
.float-add-btn {
  position: fixed;
  top: 80rpx;
  right: 30rpx;
  z-index: 999;
}
.float-add-btn button {
  background-color: #2979ff;
  color: #fff;
  font-weight: bold;
  border-radius: 50px;
  padding: 10px 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

</style>
