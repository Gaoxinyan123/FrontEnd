<template>
  <view class="list row">
    <!-- 留言列表模块(结束) -->
    <view class="list_message">
      <view class="item_message" v-for="(o,i) in list" :key="i">
        <view class="message_box">
          <view class="title">
            <view>{{o.title}}</view>
          </view>
          <view class="user_info">
            <view>{{o.nickname}}</view>
            <view class="time">{{$toTime(o.create_time,"yyyy-MM-dd hh:mm:ss")}}</view>
          </view>
          <view class="content">
            <rich-text :nodes="o.content"></rich-text>
          </view>
        </view>
        <!-- 回复内容 -->
        <view class="list_reply_block" v-if="list[i].reply_state">
			<view>回复</view>
          <view class="list_reply_box">
            <rich-text class="content" :nodes="list[i].reply"></rich-text>
          </view>
        </view>
        <!-- /回复内容 -->
        <view class="line"></view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  props: {
    list: {
      type: Array,
      default: function () {
        return [];
      },
    },
    vm: {
      type: Object,
      default: function () {
        return {
          img: "img",
          message_id: "message_id",
          nickname: "nickname",
          title: "title",
          create_time: "create_time",
          content: "content",
          praise_len: "praise_len",
          hits: "hits",
          reply:"reply",
          reply_state:"reply_state"
        };
      },
    },
    img_width: {
      type: String,
      default: "5rem",
    },
  },
  data() {
    return {
    };
  },
  mounted(){
  },
  created() {
  },
  methods: {
  },
};
</script>

<style lang="scss" scoped>
.list {
  .item {
    display: flex;
    padding: 0.5rem;
    .image {
      margin-right: 1rem;
      &:hover {
        transform: rotateY(360deg);
        transition: all 0.5s ease-in-out;
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
