<template>
  <view>
    <view class="container diy_view">
      <view>
        <view>
          <view class="">
            <uni-forms :modelValue="form"  v-if="is_view()">

              <uni-forms-item v-if="$check_field('get','publish_users') || ($check_field('add','publish_users') || $check_field('set','publish_users'))" label="发布用户" name="publish_users">
                    <uni-data-select
                  id="form_publish_users"
                  v-model="form['publish_users']"
                  :localdata="list_user_publish_users"
                  :clear="!disabledObj['publish_users_isDisabled']"
                  :disabled="disabledObj['publish_users_isDisabled']"
                  v-if="user_group === '管理员' || (form['tree_hole_release_id'] && $check_field('set','publish_users')) || (!form['tree_hole_release_id'] && $check_field('add','publish_users'))"
                ></uni-data-select>
                <uni-data-select
                  v-model="form['publish_users']"
                  :localdata="list_user_publish_users"
                  :clear="false"
                  :disabled="true"
                  v-else-if="$check_field('get','publish_users')" id="publish_users"
                ></uni-data-select>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','content_title') || ($check_field('add','content_title') || $check_field('set','content_title'))" label="内容标题" name="content_title">
                            <uni-easyinput type="text" v-model="form['content_title']" v-if="user_group === '管理员' || (form['tree_hole_release_id'] && $check_field('set','content_title')) || (!form['tree_hole_release_id'] && $check_field('add','content_title'))" :disabled="disabledObj['content_title_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','content_title')">
                  {{ form['content_title'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','content_classification') || ($check_field('add','content_classification') || $check_field('set','content_classification'))" label="内容分类" name="content_classification">
                    <ld-select :multiple="true" :list="list_content_classification"
                 label-key="text" value-key="value"
                 placeholder="请选择"
                 :clearable="!disabledObj['content_classification_isDisabled']"
                 :disabled="disabledObj['content_classification_isDisabled']"
                 v-model="content_classification_multiple_value"
                 v-if="user_group === '管理员' || (form['tree_hole_release_id'] && $check_field('set','content_classification')) || (!form['tree_hole_release_id'] && $check_field('add','content_classification'))"
                 @confirm="select_content_classification_multiple"></ld-select>
              <!-- 仅查看 -->
              <text v-else-if="$check_field('get','content_classification')">
                {{ form['content_classification'] }}
              </text>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','image_content') || ($check_field('add','image_content') || $check_field('set','image_content'))" label="图片内容" name="image_content">
                    <!-- 修改权限 -->
                <view class="diy_field diy_img" v-if="form['image_content'] && $check_field('set','image_content')">
                  <image v-if="disabledObj['image_content_isDisabled']" :src="$fullUrl(form['image_content'])" />
                  <image v-if="!disabledObj['image_content_isDisabled']" :src="$fullUrl(form['image_content'])" @click="change_img('image_content')" />
                </view>
                <!-- 添加权限 -->
                <view class="diy_field diy_img" v-else-if="!form['image_content'] && $check_field('add','image_content')">
                  <view v-if="disabledObj['image_content_isDisabled']" class="btn_add_img">
                    <text>+</text>
                  </view>
                  <view v-if="!disabledObj['image_content_isDisabled']" class="btn_add_img" @click="change_img('image_content')">
                    <text>+</text>
                  </view>
                </view>
                <!-- 查询权限 -->
                <view class="diy_field diy_img" v-else-if="$check_field('get','image_content')">
                  <image :src="$fullUrl(form['image_content'])" />
                </view>
                  </uni-forms-item>
			 <!-- 	<uni-forms-item v-if="$check_field('get','video_content') || ($check_field('add','video_content') || $check_field('set','video_content'))" label="视频内容" name="video_content">-->
                    <!-- 查询权限 -->
                <view class="diy_field diy_video" v-if="$check_field('get','video_content') && form['video_content']">
				<view class="close_" @click="close_('video_content')">x</view>
					<video
						:src="$fullUrl(form['video_content'])"
						controls
					></video>
				</view>
				 <!--<button v-else-if="$check_field('add','video_content') || $check_field('set','video_content')" class="mini-btn" type="primary" size="mini" @click="uploadFile_('video_content')">上传视频</button>-->
				<view class="file-url" v-if="video_content">{{video_content}}</view>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','text_content') || ($check_field('add','text_content') || $check_field('set','text_content'))" label="文字内容" name="text_content">
                    <uni-easyinput type="textarea" v-model="form['text_content']" v-if="user_group === '管理员' || (form['tree_hole_release_id'] && $check_field('set','text_content')) || (!form['tree_hole_release_id'] && $check_field('add','text_content'))" :disabled="disabledObj['text_content_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','text_content')">
                  {{ form['text_content'] }}
                </text>
                  </uni-forms-item>


            </uni-forms>
            <view class="form_button" v-if="$check_action('/tree_hole_release/view','set') || ($check_action('/tree_hole_release/view','add') || $check_option('/tree_hole_release/table','examine'))">
              <button size="mini" type="primary" @click="submit_()" class="primary_btn">提交</button>
              <button size="mini" @click="cancel()">取消</button>
            </view>
            <view class="form_button" v-else>
              <button size="mini" @click="cancel()">返回</button>
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import mixin from "@/libs/mixins/page.js";
                            import ldSelect from "@/components/ld-select/ld-select.vue";
export default {
  mixins: [mixin],
  components: {ldSelect},
  data() {
    return {
      field: "tree_hole_release_id",
      url_add: "~/api/tree_hole_release/add?",
      url_set: "~/api/tree_hole_release/set?",
      url_get_obj: "~/api/tree_hole_release/get_obj?",
      url_upload: "~/api/tree_hole_release/upload?"
	  ,publish_users: null
	  ,content_title: null
	  ,content_classification: null
	  ,image_content: null
	  ,video_content: null
	  ,text_content: null
      ,query: {
        "tree_hole_release_id": 0,
      },

      form: {
            "publish_users": 0, // 发布用户
                    "content_title":  '', // 内容标题
                    "content_classification":  '', // 内容分类
                    "image_content":  '', // 图片内容
                    "video_content":  '', // 视频内容
                    "text_content":  '', // 文字内容
                                    "tree_hole_release_id": 0, // ID
                
              },
          disabledObj:{
                        "publish_users_isDisabled": false,
                                "content_title_isDisabled": false,
                                "content_classification_isDisabled": false,
                                "image_content_isDisabled": false,
                                "video_content_isDisabled": false,
                                "text_content_isDisabled": false,
                                  },
                                // 用户列表
            list_user_publish_users: [],
                        // 用户组
            group_user_publish_users: "",
                                                                            content_classification_multiple_value:[],
                      // 内容分类选项列表
          list_content_classification: [],
                                                                                                      }
  },
  methods: {
    /**
     * 初始化前事件 - 确保URL参数正确设置到query对象
     */
    init_before(query) {
      // 确保tree_hole_release_id从URL参数正确设置
      if (query && query.tree_hole_release_id) {
        this.query.tree_hole_release_id = query.tree_hole_release_id;
        console.log("设置tree_hole_release_id:", this.query.tree_hole_release_id);
      }
      return query;
    },

    /**
     * 提交前验证事件
     * @param {Object} 请求参数
     * @return {String} 验证成功返回null, 失败返回错误提示
     */
    submit_check(param) {
                                                                                                      return null;
    },

    changeLog(v,value){
      this.form[value] = v
    }
	,uploadFile_(type) {
			// #ifdef APP-VUE
				uni.chooseFile({
					count: 1,
					success: (chooseImageRes) => {
						const tempFilePaths = chooseImageRes.tempFilePaths;
						this.successChoose(tempFilePaths[0], type)
					}
				});
			// #endif
			// #ifdef !APP-VUE
				if (uni.getSystemInfoSync().uniPlatform != "mp-weixin" || uni.getSystemInfoSync().platform == "devtools") {
					uni.chooseImage({
						count: 1,
						success: (chooseImageRes) => {
							const tempFilePaths = chooseImageRes.tempFilePaths;
							this.successChoose(tempFilePaths[0], type)
						}
					});
				} else {
					wx.chooseMessageFile({
						count: 1,
						success: (chooseImageRes) => {
							const tempFilePaths = chooseImageRes.tempFiles;
							this.successChoose(tempFilePaths[0].path, type)
						}
					})
				}
			// #endif

		}
		,successChoose(filePath, type) {
			uni.uploadFile({
				url: getApp().globalData.host + '/api/tree_hole_release/upload', //仅为示例，非真实的接口地址
				filePath,
				name: 'file',
				success: (uploadFileRes) => {
					if (uploadFileRes && uploadFileRes.data) {
						let dataObj;
						try {
							dataObj = JSON.parse(uploadFileRes.data);
						} catch(e) {
							uni.showToast({title: '返回数据格式错误', icon: 'none'});
							return;
						}
						if (dataObj && dataObj.result && dataObj.result.url) {
							var filename = dataObj.result.url;
							this[type] = filename;
						} else {
							uni.showToast({title: '上传失败', icon: 'none'});
						}
					} else {
						uni.showToast({title: '上传失败', icon: 'none'});
					}
				}
			});
		}
	,close_(type) {
					if (type == 'publish_users') this['publish_users'] = this.form['publish_users'] = ""
					if (type == 'content_title') this['content_title'] = this.form['content_title'] = ""
					if (type == 'content_classification') this['content_classification'] = this.form['content_classification'] = ""
					if (type == 'image_content') this['image_content'] = this.form['image_content'] = ""
					if (type == 'video_content') this['video_content'] = this.form['video_content'] = ""
					if (type == 'text_content') this['text_content'] = this.form['text_content'] = ""
			}
	,submit_() {
		// 直接使用form对象中的数据，不需要检查null值
		// 因为form对象已经包含了用户修改的数据
		console.log("提交的表单数据:", this.form)
		console.log("当前用户组:", this.user_group)
		console.log("是否有set权限:", this.$check_action('/tree_hole_release/view','set'))
		console.log("URL设置:", this.url_set)
		console.log("字段ID:", this.field)
		console.log("当前query对象:", this.query)
		console.log("form中的tree_hole_release_id:", this.form.tree_hole_release_id)
		console.log("query中的tree_hole_release_id:", this.query.tree_hole_release_id)
		
		// 确保form中有正确的ID
		if (this.query.tree_hole_release_id && this.query.tree_hole_release_id !== 0) {
			this.form.tree_hole_release_id = this.query.tree_hole_release_id;
			console.log("已设置form.tree_hole_release_id为:", this.form.tree_hole_release_id);
		}
		
		this.submit()
	}
    /**
     * 上传文件
     * @param {Object} param文件参数
     */
    ,change_file(key_name){
      var _self = this;
      // 选择图像方法
      uni.chooseFile({
        count: 1,
        sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
        sourceType: ['album'], //从相册选择
        success: function(res) {
          const tempFilePaths = res.tempFilePaths;
          const uploadTask = uni.uploadFile({
            url: _self.$fullUrl('/api/tree_hole_release/upload?'),
            filePath: tempFilePaths[0],
            name: 'file',
            formData: {
              'i_want_to_customize': 'test'
            },
            header: {
              'x-auth-token': _self.$store.state.user.token
            },
            success: function(uploadFileRes) {
              if (uploadFileRes && uploadFileRes.data) {
                let dataObj;
                try {
                  dataObj = JSON.parse(uploadFileRes.data);
                } catch(e) {
                  uni.showToast({title: '返回数据格式错误', icon: 'none'});
                  return;
                }
                if (dataObj && dataObj.result && dataObj.result.url) {
                  var filename = dataObj.result.url;
                  _self.form[key_name] = filename;
                } else {
                  uni.showToast({title: '上传失败', icon: 'none'});
                }
              } else {
                uni.showToast({title: '上传失败', icon: 'none'});
              }
            }
          });

          uploadTask.onProgressUpdate(function(res) {
            _self.percent = res.progress;
            console.log('上传进度' + res.progress);
            console.log('已经上传的数据长度' + res.totalBytesSent);
            console.log('预期需要上传的数据总长度' + res.totalBytesExpectedToSend);
          });
        },
        error: function(e) {
          console.log(e);
        }
      });
    },
    /**
     * 上传图片
     * @param {Object} param文件参数
     */
    change_img(key_name){
      var _self = this;
      _self.upload_img_flag = false
      // 选择图像方法
      uni.chooseImage({
        count: 1,
        sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
        sourceType: ['album'], //从相册选择
        success: function(res) {
          const tempFilePaths = res.tempFilePaths;
          const uploadTask = uni.uploadFile({
            url: _self.$fullUrl('/api/tree_hole_release/upload?'),
            filePath: tempFilePaths[0],
            name: 'file',
            formData: {
              'i_want_to_customize': 'test'
            },
            header: {
              'x-auth-token': _self.$store.state.user.token
            },
            success: function(uploadFileRes) {
              if (uploadFileRes && uploadFileRes.data) {
                let dataObj;
                try {
                  dataObj = JSON.parse(uploadFileRes.data);
                } catch(e) {
                  uni.showToast({title: '返回数据格式错误', icon: 'none'});
                  return;
                }
                if (dataObj && dataObj.result && dataObj.result.url) {
                  var filename = dataObj.result.url;
                  _self.form[key_name] = filename;
                } else {
                  uni.showToast({title: '上传失败', icon: 'none'});
                }
              } else {
                uni.showToast({title: '上传失败', icon: 'none'});
              }
            }
          });

          uploadTask.onProgressUpdate(function(res) {
            _self.percent = res.progress;
            console.log('上传进度' + res.progress);
            console.log('已经上传的数据长度' + res.totalBytesSent);
            console.log('预期需要上传的数据总长度' + res.totalBytesExpectedToSend);
          });
        },
        error: function(e) {
          console.log(e);
        }
      });
    },
    /**
     * 获取对象后获取缓存表单
     * @param {Object} json
     * @param {Object} func
     */
    get_obj_before(param){
      var form = uni.db.get("form");
      if (form) {
        delete(form.examine_state)
        delete(form.examine_reply)
        this.obj = uni.push(this.obj ,form);
        this.form = uni.push(this.form ,form);
      }
      var arr = []
      for (let key in form) {
        arr.push(key)
      }
      for (var i=0;i<arr.length;i++){
        this.disabledObj[arr[i] + '_isDisabled'] = true
      }
                                                      uni.db.del("form");
      return param;
    },
            /**
     * 获取注册用户用户列表
     */
    async get_list_user_publish_users() {
      // if(this.user_group !== "管理员" && this.form["publish_users"] === 0) {
      //     this.form["publish_users"] = this.user.user_id;
      // }
      var json = await this.$get("~/api/user/get_list?user_group=注册用户");
      if(json.result && json.result.list){
        json.result.list.map((o) => this.list_user_publish_users.push({value:o.user_id,text:o.nickname + '-' + o.username}));
      }
      else if(json.error){
        console.error(json.error);
      }
    },
            /**
     * 获取注册用户用户组
     */
    async get_group_user_publish_users() {
      this.form["publish_users"] = this.user.user_id;
      var json = await this.$get("~/api/user_group/get_obj?name=注册用户");
      if(json.result && json.result.obj){
        this.group_user_publish_users = json.result.obj;
        this.get_user_session_publish_users(this.form['publish_users'])
      }
      else if(json.error){
        console.error(json.error);
      }
    },
    get_user_session_publish_users(id){
      var _this = this;
      var user_id = {"user_id":id}
      var url = "~/api/"+_this.group_user_publish_users.source_table+"/get_obj?"
      this.$get(url, user_id, function(res) {
        if (res && res.result && res.result.obj) {
          let o = res.result.obj;
          var arr = []
          for (let key in o) {
            arr.push(key)
          }
          var arrForm = []
          for (let key in _this.form) {
            arrForm.push(key)
          }
          _this.form["publish_users"] = id
          _this.disabledObj['publish_users' + '_isDisabled'] = true
          for (var i=0;i<arr.length;i++){
            if (arr[i]!=='examine_state' && arr[i]!=='examine_reply') {
              for (var j = 0; j < arrForm.length; j++) {
                if (arr[i] === arrForm[j]) {
                  if (arr[i] !== "publish_users") {
                    _this.form[arrForm[j]] = o[arr[i]]
                    _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                    break;
                  } else {
                    _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                  }
                }
              }
            }
          }
        } else {
          uni.showToast({title: '数据获取失败', icon: 'none'});
        }
      });
    },
            
            
            /**
     * 获取内容分类列表
     */
    async get_list_content_classification() {
                          var json = await this.$get("~/api/content_classification/get_list?");
          if(json.result && json.result.list){
            json.result.list.map((o) => this.list_content_classification.push({value:o.content_classification,text:o.content_classification}));
          }
          else if(json.error){
            console.error(json.error);
          }
            },
              select_content_classification_multiple(v){
        this.form.content_classification = "";
        if (v && v.length > 0) {
            this.form.content_classification = v.toString();
        }
      },
            
            
            
            
    
    /**
     * 获取对象之后
     * @param {Object} json
     * @param {Object} func
     */
    get_obj_after(json, func){
                                          if (this.form.content_classification){
        this.content_classification_multiple_value = this.form.content_classification.split(",")
      }
                                            },

    is_view(){
      var bl = this.user_group == "管理员";

      if(!bl){
        bl = this.$check_action('/tree_hole_release/table','add');
        console.log(bl ? "你有表格添加权限视作有添加权限" : "你没有表格添加权限");
      }
      if(!bl){
        bl = this.$check_action('/tree_hole_release/table','set');
        console.log(bl ? "你有表格添加权限视作有修改权限" : "你没有表格修改权限");
      }
      if(!bl){
        bl = this.$check_action('/tree_hole_release/view','add');
        console.log(bl ? "你有视图添加权限视作有添加权限" : "你没有视图添加权限");
      }
      if(!bl){
        bl = this.$check_action('/tree_hole_release/view','set');
        console.log(bl ? "你有视图修改权限视作有修改权限" : "你没有视图修改权限");
      }
      if(!bl){
        bl = this.$check_action('/tree_hole_release/view','get');
        console.log(bl ? "你有视图查询权限视作有查询权限" : "你没有视图查询权限");
      }

      console.log(bl ? "具有当前页面的查看权，请注意这不代表你有字段的查看权" : "无权查看当前页，请注意即便有字段查询权限没有页面查询权限也不行");

      return bl;
    },

  },
  created() {
            this.get_list_user_publish_users();
            this.get_group_user_publish_users();
                            this.get_list_content_classification();
                                  },
}
</script>

<style scoped>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}

.form_button{
  padding-bottom: 15px;
  display: flex;
}
.form_button button{
  width: 40%;
}
.query_select{
  border-color: rgb(229, 229, 229);
  background-color: rgb(255, 255, 255);
  border-radius: 4px;
  box-sizing: border-box;
  flex: 1;
  width: 100%;
  line-height: 2;
  font-size: 14px;
  height: 2.4em;
  min-height: 2.4em;
  display: block;
  outline:none;
}

.query_option{
  width: 100%;
}

.btn_add_img{
  color: #D3D3D3;
  text-align: center;
  border: 1px solid #eee;
  height: 5rem;
  width: 5rem;
  position: relative;
}
.btn_add_img text{
  font-size: 35px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}

.form_button {
  padding-bottom: 15px;
  display: flex;
}
.form_button button {
  width: 40%;
}
.query_select {
  border-color: rgb(229, 229, 229);
  background-color: rgb(255, 255, 255);
  border-radius: 4px;
  box-sizing: border-box;
  flex: 1;
  width: 100%;
  line-height: 2;
  font-size: 14px;
  height: 2.4em;
  min-height: 2.4em;
  display: block;
  outline: none;
}

.query_option {
  width: 100%;
}

.btn_add_img {
  color: #d3d3d3;
  text-align: center;
  border: 1px solid #eee;
  height: 5rem;
  width: 5rem;
  position: relative;
}
.btn_add_img text {
  font-size: 35px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
/*新样式*/
.uni-forms{
padding-top:1rem;
}
.uni-forms-item {
	padding: 6px 10px;
    background: #f8f6fc;
}
.uni-forms .is-input-border{
	border: 0;
}
.container{
	    -webkit-box-shadow: 0px 0px 0px #888888;
	    box-shadow: 0px 0px 0px #888888;
}
.form_button .primary_btn{
		background-color: #22B8B8;
		color: #FFFFFF;
	}
.file-url {
	font-size: 12px;
	color: #ccc;
}
	.diy_field, .file-url {
		position: relative;
	}
	.close_ {
		position: absolute;
		top: -18px;
		left: -7px;
		font-size: 22px;
		color: #22B8B8;
		font-weight: 600;
	}


</style>
