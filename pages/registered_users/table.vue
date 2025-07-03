<template>
  <view class="page_registered_users diy_table diy_table_tml" id="registered_users_table">
    <!-- 筛选模块(开始) -->
    <view>
      <view class="container">
        <view>
          <view>
            <view class="">
              <!-- 搜索栏 -->
              <uni-forms :modelValue="query">
                <uni-forms-item label="手机号码" name="mobile_phone_number">
                  <uni-easyinput type="text" v-model="query.mobile_phone_number" placeholder="手机号码" />
                </uni-forms-item>
              </uni-forms>
              <!-- /搜索栏 -->
              <view class="top-buttons">
                <button class="btn-top" @click="reset()">重置</button>
                <button class="btn-top" type="primary" @click="search_()">查询</button>
                <navigator class="btn-top" url="/pages/registered_users/view?"
                  v-if="is_editable() && ($check_action('/registered_users/table','add') || $check_action('/registered_users/view','add'))">
                  添加
                </navigator>
              </view>
            </view>
          </view>
        </view>
      </view>
    </view>

    <view>
      <view class="container">
        <view>
          <view>
            <!-- 列表 -->
            <view class="warp">
              <view class="container">
                <view class="row-wrap">
                  <view v-for="(o, index) in list" :key="index" class="card-item">

                    <!-- 昵称 -->
                    <view class="view center-text">
                      <div class="nickname-text">{{ o.nickname || '暂无' }}</div>
                    </view>

                    <!-- 手机号 -->
                    <view v-if="$check_field('get','mobile_phone_number')" class="view center-text">
                      <div class="phone-text">{{ o['mobile_phone_number'] }}</div>
                    </view>

                    <!-- 创建时间 -->
                    <view class="center-text">
                      {{ $toTime(o["create_time"] ,"yyyy-MM-dd hh:mm:ss") }}
                    </view>

                    <!-- 按钮行 -->
                    <view class="button-row-center" v-if="is_editable()">
                      <navigator :url="'/pages/registered_users/view?' + field + '=' + o[field]"
                        v-if="$check_action('/registered_users/view','get')"
                        class="btn-action btn-lg">
                        详情
                      </navigator>
                      <button class="btn-action danger btn-lg" @click="delInfo(index)"
                        v-if="$check_action('/registered_users/view','del')">
                        删除
                      </button>
                    </view>

                  </view>
                </view>
              </view>
            </view>
            <!-- /列表 -->
          </view>
        </view>
      </view>
    </view>

    <!-- 分页器 -->
    <uni-pagination class="pager" show-icon="true" :total="count" :pageSize="query.size"
      :current="query.page" @change="page_change">
    </uni-pagination>
    <!-- /分页器 -->

  </view>
</template>

<script>
import mixin from "@/libs/mixins/page.js";

export default {
  mixins: [mixin],
  data() {
    return {
      url_get_list: "~/api/registered_users/get_list?like=0",
      url_del: "~/api/registered_users/del?",
      field: "registered_users_id",
      query: {
        "size": 7,
        "page": 1,
        "mobile_phone_number": "",
        "login_time": "",
        "create_time": "",
      },
      list: [],
    };
  },
  computed: {
    groupedList() {
      const arr = [];
      for (let i = 0; i < this.list.length; i += 2) {
        arr.push(this.list.slice(i, i + 2));
      }
      return arr;
    },
  },
  methods: {
    is_editable() {
      return this.user_group === '管理员';
    },
    search_() {
      this.query.page = 1;
      this.get_list();
    },
    reset() {
      uni.clear(this.query);
      uni.push(this.query, this.config);
      this.get_list();
    },
    delInfo(v) {
      let _this = this;
      uni.showModal({
        title: '删除',
        content: '此操作将永久删除该文件, 是否继续?',
        success: function (res) {
          if (res.confirm) {
            let list = [v];
            _this.delInfoSub(list);
          }
        }
      });
    },
    async delInfoSub(list) {
      let _this = this;
      await this.delAll(list, async (list) => {
        var bl = true;
        for (var i = 0; i < list.length; i++) {
          var user_id = _this.list[list[i]].user_id;
          var res = await this.$get("~/api/user/del?", { user_id });
          if (!res.result) {
            bl = false;
            break;
          }
        }
        if (bl) {
          _this.$toast("删除成功!", 'success');
          this.get_list();
        }
      });
    },
    async get_list_after(param) {
      console.log("✅ 初始接口数据:", param);
      this.list.sort((a, b) => {
        const timeA = Date.parse(a.create_time?.toString().replace(/-/g, '/')) || 0;
        const timeB = Date.parse(b.create_time?.toString().replace(/-/g, '/')) || 0;
        return timeB - timeA;
      });
      console.log("✅ 排序后 list:", this.list);

      for (const item of this.list) {
        try {
          console.log("🔍 查询 user_id:", item.user_id);
          const res = await this.$get('~/api/user/get_list?', { user_id: item.user_id });
          console.log("📦 获取 user 数据:", res);
          if (res.result && res.result.list && res.result.list.length > 0) {
            item.nickname = res.result.list[0].nickname || '暂无';
          } else {
            item.nickname = '暂无';
          }
        } catch (e) {
          console.error("❌ 获取昵称失败：", e);
          item.nickname = '加载失败';
        }
      }
      console.log("✅ 添加 nickname 后:", this.list);

      this.list = [...this.list];
    },
    getCardBgClass(o) {
      if (o.examine_state === '已通过' || o.state === 2) {
        return 'card-bg-dark';
      } else {
        return 'card-bg-light';
      }
    },
  },
};
</script>

<style scoped>
.row-wrap {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1rem;
  margin-top: 1rem;
}

.card-item {
  width: 90%;
  background-color: #ffffff;
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  border: 1px solid #ddd;
  text-align: center;
  transition: all 0.2s;
}
.card-item:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.1);
}

.center-text {
  text-align: center;
  margin-bottom: 10px;
}

.nickname-text {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}

.phone-text {
  font-size: 14px;
  color: #666;
}

.top-buttons {
  display: flex;
  justify-content: space-between;
  padding: 10px 15px;
  gap: 10px;
}

.btn-top {
  flex: 1;
  text-align: center;
  padding: 8px 0;
  border-radius: 8px;
  background-color: #4fc08d;
  color: white;
  font-weight: bold;
  border: none;
}

.button-row-center {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-top: 12px;
}

.btn-action {
  padding: 12px 24px;
  background-color: #4fc08d;
  border-radius: 8px;
  color: white;
  font-size: 16px;
  font-weight: bold;
}

.btn-action.danger {
  background-color: #ff6666;
}
</style>