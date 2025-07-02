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
                  v-if="user_group === '管理员' || (form['content_reporting_id'] && $check_field('set','publish_users')) || (!form['content_reporting_id'] && $check_field('add','publish_users'))"
                ></uni-data-select>
                <uni-data-select
                  v-model="form['publish_users']"
                  :localdata="list_user_publish_users"
                  :clear="false"
                  :disabled="true"
                  v-else-if="$check_field('get','publish_users')" id="publish_users"
                ></uni-data-select>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','report_users') || ($check_field('add','report_users') || $check_field('set','report_users'))" label="举报用户" name="report_users">
                    <uni-data-select
                  id="form_report_users"
                  v-model="form['report_users']"
                  :localdata="list_user_report_users"
                  :clear="!disabledObj['report_users_isDisabled']"
                  :disabled="disabledObj['report_users_isDisabled']"
                  v-if="user_group === '管理员' || (form['content_reporting_id'] && $check_field('set','report_users')) || (!form['content_reporting_id'] && $check_field('add','report_users'))"
                ></uni-data-select>
                <uni-data-select
                  v-model="form['report_users']"
                  :localdata="list_user_report_users"
                  :clear="false"
                  :disabled="true"
                  v-else-if="$check_field('get','report_users')" id="report_users"
                ></uni-data-select>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','content_title') || ($check_field('add','content_title') || $check_field('set','content_title'))" label="内容标题" name="content_title">
                            <uni-easyinput type="text" v-model="form['content_title']" v-if="user_group === '管理员' || (form['content_reporting_id'] && $check_field('set','content_title')) || (!form['content_reporting_id'] && $check_field('add','content_title'))" :disabled="disabledObj['content_title_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','content_title')">
                  {{ form['content_title'] }}
                </text>
                          </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','report_type') || ($check_field('add','report_type') || $check_field('set','report_type'))" label="举报类型" name="report_type">
                    <uni-data-select
                  v-model="form.report_type"
                  :localdata="list_report_type"
                  :clear="!disabledObj['report_type_isDisabled']"
                  :disabled="disabledObj['report_type_isDisabled']"
                  v-if="user_group === '管理员' || (form['content_reporting_id'] && $check_field('set','report_type')) || (!form['content_reporting_id'] && $check_field('add','report_type'))"
                ></uni-data-select>
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','report_type')">
                  {{ form['report_type'] }}
                </text>
                  </uni-forms-item>
              <uni-forms-item v-if="$check_field('get','reporting_details') || ($check_field('add','reporting_details') || $check_field('set','reporting_details'))" label="举报详情" name="reporting_details">
                    <uni-easyinput type="textarea" v-model="form['reporting_details']" v-if="user_group === '管理员' || (form['content_reporting_id'] && $check_field('set','reporting_details')) || (!form['content_reporting_id'] && $check_field('add','reporting_details'))" :disabled="disabledObj['reporting_details_isDisabled']" />
                <!-- 仅查看 -->
                <text v-else-if="$check_field('get','reporting_details')">
                  {{ form['reporting_details'] }}
                </text>
                  </uni-forms-item>
              <uni-forms-item label="审核状态" name="examine_state">
                <uni-data-select
                    v-model="form['examine_state']"
                    :localdata="list_examine_state"
                    v-if="user_group === '管理员' || (form['examine_state'] && $check_examine()) || (!form['examine_state'] && $check_examine())"
                ></uni-data-select>
                <text v-else>{{form["examine_state"]}}</text>
              </uni-forms-item>
              <uni-forms-item label="审核回复" name="examine_reply">
                <uni-easyinput type="text" placeholder="请输入审核回复" v-model="form['examine_reply']"
                               v-if="user_group === '管理员' || (form['examine_reply'] && $check_examine()) || (!form['examine_reply'] && $check_examine())" />
                <!-- 仅查看 -->
                <text v-else>{{form["examine_reply"]}}</text>
              </uni-forms-item>


            </uni-forms>
            <view class="form_button" v-if="$check_action('/content_reporting/view','set') || ($check_action('/content_reporting/view','add') || $check_option('/content_reporting/table','examine'))">
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
                    export default {
  mixins: [mixin],
  data() {
    return {
      field: "content_reporting_id",
      url_add: "~/api/content_reporting/add?",
      url_set: "~/api/content_reporting/set?",
      url_get_obj: "~/api/content_reporting/get_obj?",
      url_upload: "~/api/content_reporting/upload?"
	  ,publish_users: null
	  ,report_users: null
	  ,content_title: null
	  ,report_type: null
	  ,reporting_details: null
      ,query: {
        "content_reporting_id": 0,
      },

      form: {
            "publish_users": 0, // 发布用户
                    "report_users": 0, // 举报用户
                    "content_title":  '', // 内容标题
                    "report_type":  '', // 举报类型
                    "reporting_details":  '', // 举报详情
                        "examine_state": "未审核",
                    "examine_reply": "",
                        "content_reporting_id": 0, // ID
                
              },
          disabledObj:{
                        "publish_users_isDisabled": false,
                                "report_users_isDisabled": false,
                                "content_title_isDisabled": false,
                                "report_type_isDisabled": false,
                                "reporting_details_isDisabled": false,
                                  },
                                // 用户列表
            list_user_publish_users: [],
                                                    // 用户列表
            list_user_report_users: [],
                        // 用户组
            group_user_report_users: "",
                                                                          // 举报类型选项列表
          list_report_type: [],
                                                            list_examine_state:[{value:"未审核",text:"未审核"},{value:"已通过",text:"已通过"},{value:"未通过",text:"未通过"}],
                  }
  },
  methods: {
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
				url: getApp().globalData.host + '/api/content_reporting/upload', //仅为示例，非真实的接口地址
				filePath,
				name: 'file',
				success: (uploadFileRes) => {
					if (uploadFileRes.data.error) {
						uni.showToast({title: uploadFileRes.data.error.message, icon: "none"})
					} else {
						this[type] = JSON.parse(uploadFileRes.data).result.url
					}
				}
			});
		}
	,close_(type) {
					if (type == 'publish_users') this['publish_users'] = this.form['publish_users'] = ""
					if (type == 'report_users') this['report_users'] = this.form['report_users'] = ""
					if (type == 'content_title') this['content_title'] = this.form['content_title'] = ""
					if (type == 'report_type') this['report_type'] = this.form['report_type'] = ""
					if (type == 'reporting_details') this['reporting_details'] = this.form['reporting_details'] = ""
			}
	,submit_() {
					if (this['publish_users'] !== null) this.form['publish_users'] = this['publish_users']
					if (this['report_users'] !== null) this.form['report_users'] = this['report_users']
					if (this['content_title'] !== null) this.form['content_title'] = this['content_title']
					if (this['report_type'] !== null) this.form['report_type'] = this['report_type']
					if (this['reporting_details'] !== null) this.form['reporting_details'] = this['reporting_details']
				console.log(this.form)
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
            url: _self.$fullUrl('/api/content_reporting/upload?'),
            filePath: tempFilePaths[0],
            name: 'file',
            formData: {
              'i_want_to_customize': 'test'
            },
            header: {
              'x-auth-token': _self.$store.state.user.token
            },
            success: function(uploadFileRes) {
              var filename = JSON.parse(uploadFileRes.data).result.url
              _self.form[key_name] = filename
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
            url: _self.$fullUrl('/api/content_reporting/upload?'),
            filePath: tempFilePaths[0],
            name: 'file',
            formData: {
              'i_want_to_customize': 'test'
            },
            header: {
              'x-auth-token': _self.$store.state.user.token
            },
            success: function(uploadFileRes) {
              var filename = JSON.parse(uploadFileRes.data).result.url
              _self.form[key_name] = filename
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
     * 获取注册用户用户列表
     */
    async get_list_user_report_users() {
      // if(this.user_group !== "管理员" && this.form["report_users"] === 0) {
      //     this.form["report_users"] = this.user.user_id;
      // }
      var json = await this.$get("~/api/user/get_list?user_group=注册用户");
      if(json.result && json.result.list){
        json.result.list.map((o) => this.list_user_report_users.push({value:o.user_id,text:o.nickname + '-' + o.username}));
      }
      else if(json.error){
        console.error(json.error);
      }
    },
            /**
     * 获取注册用户用户组
     */
    async get_group_user_report_users() {
      this.form["report_users"] = this.user.user_id;
      var json = await this.$get("~/api/user_group/get_obj?name=注册用户");
      if(json.result && json.result.obj){
        this.group_user_report_users = json.result.obj;
        this.get_user_session_report_users(this.form['report_users'])
      }
      else if(json.error){
        console.error(json.error);
      }
    },
    get_user_session_report_users(id){
      var _this = this;
      var user_id = {"user_id":id}
      var url = "~/api/"+_this.group_user_report_users.source_table+"/get_obj?"
      this.$get(url, user_id, function(res) {
        if (res.result && res.result.obj) {
          var arr = []
          for (let key in res.result.obj) {
            arr.push(key)
          }
          var arrForm = []
          for (let key in _this.form) {
            arrForm.push(key)
          }
          _this.form["report_users"] = id
          _this.disabledObj['report_users' + '_isDisabled'] = true
          for (var i=0;i<arr.length;i++){
            if (arr[i]!=='examine_state' && arr[i]!=='examine_reply') {
              for (var j = 0; j < arrForm.length; j++) {
                if (arr[i] === arrForm[j]) {
                  if (arr[i] !== "report_users") {
                    _this.form[arrForm[j]] = res.result.obj[arr[i]]
                    _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                    break;
                  } else {
                    _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                  }
                }
              }
            }
          }
        }
      });
    },
            
            
            /**
     * 获取举报类型列表
     */
    async get_list_report_type() {
                  ['违反法律法规','谣言及不实信息','违规推广','不友善行为','违反公序良俗','其他'].map((o) => this.list_report_type.push({value:o,text:o}));
                    },
                
            
    
    /**
     * 获取对象之后
     * @param {Object} json
     * @param {Object} func
     */
    get_obj_after(json, func){
                                                                },

    is_view(){
      var bl = this.user_group == "管理员";

      if(!bl){
        bl = this.$check_action('/content_reporting/table','add');
        console.log(bl ? "你有表格添加权限视作有添加权限" : "你没有表格添加权限");
      }
      if(!bl){
        bl = this.$check_action('/content_reporting/table','set');
        console.log(bl ? "你有表格添加权限视作有修改权限" : "你没有表格修改权限");
      }
      if(!bl){
        bl = this.$check_action('/content_reporting/view','add');
        console.log(bl ? "你有视图添加权限视作有添加权限" : "你没有视图添加权限");
      }
      if(!bl){
        bl = this.$check_action('/content_reporting/view','set');
        console.log(bl ? "你有视图修改权限视作有修改权限" : "你没有视图修改权限");
      }
      if(!bl){
        bl = this.$check_action('/content_reporting/view','get');
        console.log(bl ? "你有视图查询权限视作有查询权限" : "你没有视图查询权限");
      }

      console.log(bl ? "具有当前页面的查看权，请注意这不代表你有字段的查看权" : "无权查看当前页，请注意即便有字段查询权限没有页面查询权限也不行");

      return bl;
    },

  },
  created() {
            this.get_list_user_publish_users();
                        this.get_list_user_report_users();
            this.get_group_user_report_users();
                            this.get_list_report_type();
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
