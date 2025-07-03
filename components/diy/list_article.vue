<template>
  <view class="list row">
    <navigator
      v-for="(o, i) in list"
      :key="i"
      class="item"
      :url="'/pages/article/details?' + vm.article_id + '=' + o[vm.article_id]"
    >
      <image
        class="image"
        :style="{ width: img_width, height: img_width }"
        :src="$fullImgUrl(o[vm.img])"
        mode="scaleToFill"
      >
      </image>
      <view class="right_block">
        <view class="top_info">
          {{ o[vm.title] }}
        </view>
        <view class="mid_info">{{
          $toTime(o[vm.create_time], "yyyy-MM-dd hh:mm:ss")
        }}</view>
        <view class="bottom_info">
          <text class="praise">{{ o[vm.praise_len] }}点赞</text>
          <text class="see"> {{ o[vm.hits] }}点击 </text>
        </view>
        <!-- 在每个文章项中添加如下 -->
<view v-if="current_user.user_group === '管理员'" class="delete-btn">
  <button @click.stop="deleteArticle(o)">删除</button>
</view>

      </view>
    </navigator>
  </view>
</template>


<script>
export default {
  props: {
    list: {
      type: Array,
      default: () => [],
    },
    vm: {
      type: Object,
      default: () => ({
        img: "img",
        article_id: "article_id",
        title: "title",
        description: "description",
        create_time: "create_time",
        content: "content",
        praise_len: "praise_len",
        hits: "hits",
      }),
    },
    img_width: {
      type: String,
      default: "5rem",
    },
    current_user: {
      type: Object,
      default: () => ({}),
    },
  },
  methods: {
    deleteArticle(article) {
      uni.showModal({
        title: '确认删除',
        content: '确定删除该文章？',
        success: (res) => {
          if (res.confirm) {
            this.$post('~/api/article/del?', { article_id: article.article_id }, (json) => {
              if (json.result) {
                this.$emit('delete');
                uni.showToast({ title: '删除成功', icon: 'success' });
              } else {
                uni.showToast({ title: '删除失败', icon: 'none' });
              }
            });
          }
        },
      });
    },
  }
};
</script>


<style lang="scss" scoped>
.list {
  .item {
    display: flex;
    padding: 0.5rem;
    &:hover {
      background: #e0e0e0;
    }
    .image {
      margin-right: 1rem;
      transition: transform 0.3s ease;
      &:hover {
        transform: scale(1.05);
      }
    }
    .right_block {
      flex: 1;
      display: flex;
      flex-direction: column;
	  line-height: normal;
      .top_info {
        flex: 1;
        font-size: 0.8rem;
        color: $uni-text-color;
      }
      .mid_info {
		  flex: 1;
        font-size: 0.5rem;
        color: $uni-text-color-grey;
      }
      .bottom_info {
		flex: 1;
        display: flex;
        justify-content: space-between;
        align-items: baseline;
        font-size: 0.5rem;
        color: $uni-text-color-grey;
      }
    }
  }
}
</style>
