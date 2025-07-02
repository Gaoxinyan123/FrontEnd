<template>
	<view class="diy_edit page_content_reporting" id="content_reporting_edit">
		<view class='warp'>
			<view class='container'>
				<view class='row'>
							<view v-if="$check_field('set','publish_users') || $check_field('add','publish_users') || $check_field('get','publish_users')" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								发布用户:
							</text>
						</view>
						<view class="diy_field diy_down diy_text_row diy_select_flex">
							<uni-data-select
									id="form_publish_users"
									v-model="form['publish_users']"
									:localdata="list_user_publish_users"
									:clear="!disabledObj['publish_users_isDisabled']"
									:disabled="disabledObj['publish_users_isDisabled']"
									v-if="(form['publish_users'] && $check_field('set','publish_users')) || (!form['publish_users'] && $check_field('add','publish_users'))"
							></uni-data-select>
							<text v-else-if="$check_field('get','publish_users')">{{ form['publish_users'] }}</text>
						</view>
					</view>
									<view v-if="$check_field('set','report_users') || $check_field('add','report_users') || $check_field('get','report_users')" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								举报用户:
							</text>
						</view>
						<view class="diy_field diy_down diy_text_row diy_select_flex">
							<uni-data-select
									id="form_report_users"
									v-model="form['report_users']"
									:localdata="list_user_report_users"
									:clear="!disabledObj['report_users_isDisabled']"
									:disabled="disabledObj['report_users_isDisabled']"
									v-if="(form['report_users'] && $check_field('set','report_users')) || (!form['report_users'] && $check_field('add','report_users'))"
							></uni-data-select>
							<text v-else-if="$check_field('get','report_users')">{{ form['report_users'] }}</text>
						</view>
					</view>
									<view v-if="$check_field('set','content_title') || $check_field('add','content_title') || $check_field('get','content_title')" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								内容标题:
							</text>
						</view>
								<!-- 文本 -->
									<view class="diy_field diy_text diy_text_row">
							<input type="text" id="form_content_title" v-model="form['content_title']" placeholder="请输入内容标题" v-if="(form['content_title'] && $check_field('set','content_title')) || (!form['content_title'] && $check_field('add','content_title'))" :disabled="disabledObj['content_title_isDisabled']" />
							<text v-else-if="$check_field('get','content_title')">{{ form['content_title'] }}</text>
						</view>
										</view>
								<view v-if="$check_field('set','report_type') || $check_field('add','report_type') || $check_field('get','report_type')" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								举报类型:
							</text>
						</view>
								<!-- 选项 -->
						<view class="diy_field diy_down diy_text_row diy_select_flex">
							<uni-data-select
									id="form_report_type"
									v-model="form['report_type']"
									:localdata="list_report_type"
									v-if="(form['report_type'] && $check_field('set','report_type')) || (!form['report_type'] && $check_field('add','report_type'))"
							></uni-data-select>
							<text v-else-if="$check_field('get','report_type')">{{ form['report_type'] }}</text>
						</view>
							</view>
								<view v-if="$check_field('set','reporting_details') || $check_field('add','reporting_details') || $check_field('get','reporting_details')" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								举报详情:
							</text>
						</view>
								<!-- 多文本 -->
						<view class="diy_field diy_desc diy_text_row">
							<textarea id="form_reporting_details" v-model="form['reporting_details']" v-if="(form['reporting_details'] && $check_field('set','reporting_details')) || (!form['reporting_details'] && $check_field('add','reporting_details'))" :disabled="disabledObj['reporting_details_isDisabled']"/>
							<text v-else-if="$check_field('get','reporting_details')">{{ form['reporting_details'] }}</text>
						</view>
							</view>
						<view v-if="user_group === '管理员' || $check_examine()" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								审核状态:
							</text>
						</view>
						<view class="diy_field diy_text diy_text_row diy_select_flex">
							<uni-data-select
									v-model="form['examine_state']"
									:localdata="list_examine_state"
							></uni-data-select>
						</view>
						<view class="diy_field diy_text diy_text_row">
							<text>
								{{ form['examine_state'] }}
							</text>
						</view>
					</view>
					<view v-if="user_group === '管理员' || $check_examine()" class="col-12 col-md-6 row-item">
						<view class="diy_title diy_text_row">
							<text>
								审核回复:
							</text>
						</view>
						<view class="diy_field diy_text diy_text_row">
							<textarea v-model="form['examine_reply']">
							</textarea>
						</view>
						<view class="diy_field diy_text diy_text_row">
							<text>
								{{ form['examine_reply'] }}
							</text>
						</view>
					</view>

				</view>
				<view class="row">
					<view class="col-12">
						<view class="btn_box">
							<button class="btn_submit primary_btn" @click="submit()">提交</button>
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
	mixins:[mixin],
	data(){
		return{
			url_get_obj:"~/api/content_reporting/get_obj?",
			url_add:"~/api/content_reporting/add?",
			url_set:"~/api/content_reporting/set?",

			// 登录权限
			oauth: {
				"signIn": true,
				"user_group": []
			},

			// 查询条件
			query: {
					"publish_users": 0,
						"report_users": 0,
						"content_title": "",
						"report_type": "",
						"reporting_details": "",
					"content_reporting_id": 0
			},

			obj: {
					"publish_users": 0, // 发布用户
						"report_users": 0, // 举报用户
						"content_title":  '', // 内容标题
						"report_type":  '', // 举报类型
						"reporting_details":  '', // 举报详情
					"examine_state": "未审核",
				"examine_reply": "",
				"content_reporting_id": 0,

			},

			// 表单字段
			form: {
					"publish_users": 0, // 发布用户
						"report_users": 0, // 举报用户
						"content_title":  '', // 内容标题
						"report_type":  '', // 举报类型
						"reporting_details":  '', // 举报详情
					"examine_state": "未审核",
				"examine_reply": "",
				"content_reporting_id": 0,
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
									list_report_type: [],
				
			field:"content_reporting_id",
			table_key:"content_reporting",

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
				async get_user_session_report_users(){
			var _this = this;
			var json = await this.$get("~/api/user_group/get_obj?name=注册用户");
			if(json.result && json.result.obj){
				var source_table = json.result.obj.source_table;
				var user_id = _this.$store.state.user.user_id;
				if (user_id){
					var url = "~/api/"+source_table+"/get_obj?"
					this.$get(url, {"user_id":_this.$store.state.user.user_id}, function(res) {
						if (res.result && res.result.obj) {
							var arr = []
							for (let key in res.result.obj) {
								arr.push(key)
							}
							var arrForm = []
							for (let key in _this.form) {
								arrForm.push(key)
							}
							_this.form["report_users"] = user_id
							_this.disabledObj['report_users' + '_isDisabled'] = true
							for (var i=0;i<arr.length;i++){
                if (arr[i]!=='examine_state' && arr[i]!=='examine_reply') {
                  for (var j = 0; j < arrForm.length; j++) {
                    if (arr[i] === arrForm[j]) {
                      if (arr[i] !== "report_users") {
                        _this.form[arrForm[j]] = res.result.obj[arr[i]]
                        _this.disabledObj[arrForm[j] + '_isDisabled'] = true
                        break;
                      }
                    }
                  }
                }
							}
						}
					});
				}
			}
			else if(json.error){
				console.error(json.error);
			}
		},
	
	
				
	
				/**
		 * 获取举报类型列表
		 */
		async get_list_report_type() {
					['违反法律法规','谣言及不实信息','违规推广','不友善行为','违反公序良俗','其他'].map((o) => this.list_report_type.push({value:o,text:o}));
						},
							
	
				
	
			change_file(key_name){
			var _self = this;
				this.$chooseFile().then(res=>{
					console.log(res)

						const uploadTask = uni.uploadFile({
							url: _self.$fullUrl("/api/feedback/upload?"),
							filePath: res[0].path,
							name: "file",
							formData: {
								i_want_to_customize: "test",
							},
							header: {
								"x-auth-token": _self.$store.state.user.token,
							},
							success: function(uploadFileRes) {
								console.log(uploadFileRes)
								var filename = JSON.parse(uploadFileRes.data).result.url;
								_self.form[key_name] = filename;
							},
						});

						uploadTask.onProgressUpdate(function(res) {
							_self.percent = res.progress;
							console.log("上传进度" + res.progress);
							console.log("已经上传的数据长度" + res.totalBytesSent);
							console.log(
								"预期需要上传的数据总长度" + res.totalBytesExpectedToSend
							);
						});

				})
		},
		change_img(key_name) {
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
							'content_reporting': 'test'
						},
						header: {
							'x-auth-token': _self.$store.state.user.token
						},
						success: function(uploadFileRes) {
							var filename = JSON.parse(uploadFileRes.data).result.url
							var img_url = filename
							_self.form[key_name] = img_url
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
		 * 获取对象后获取缓存表单
		 * @param {Object} json
		 * @param {Object} func
		 */
		get_obj_after(json ,func){
			var form = uni.db.get("form");
			var obj = Object.assign({} ,form ,this.obj);
			if (form) {
				this.obj = uni.push(this.obj ,obj);
			}
			if (form) {
				this.form = uni.push(this.form ,form);
			}
			if(func){
				func(json);
			}
		},

	},
	onLoad(){
					this.get_list_user_publish_users();
					this.get_user_session_report_users();
				this.get_list_user_report_users();
							this.get_list_report_type();
							},
}
</script>

<style scoped>
	input{
		font-size: 10px;
	}

	.form_edit {
		background-color: #fff;
		margin-bottom: 0.5rem;
		padding: 1rem;
		font-size: 10px;
	}

	.item {
		display: flex;
		padding: 0.2rem 0;
	}

	.left_text {
		flex: 0 0 25%;
		display: flex;
		align-items: center;
	}

	.right_text {
		flex: 0 0 75%;
		border-bottom: 1px solid #eee;
	}
	.right_text.btn_warp{
		border-bottom: 0;
	}

	.btn_submit {
		text-align: center;
		background-color: #fff;
		padding: 0.3rem;
		margin: 0.1rem 1rem;
		border: 1px solid #eee;
		border-radius: 0.5rem;
	}

	.btn_submit:hover {
		opacity: 0.5;
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




</style>
<style scoped>
/*新样式*/
.diy_text_row {
  display: inline-block;
}
.container {
	margin-top:1rem ;
	padding: 1rem;
    -webkit-box-shadow: 0px 0px 0px #888888;
    box-shadow: 0px 0px 0px #888888;
}
.primary_btn{
		background-color: #22B8B8;
		color: #FFFFFF;
	}
	.btn_submit{
		padding: 0;
		margin-top:1rem ;
	}
	.row-item {
		padding: 10px 10px;
	    background: #f8f6fc;
		margin-bottom: 1rem;
	}
	.diy_field{
		padding-left: 1rem;
	}
	.diy_title{
		align-items: center;
        font-size: 14px;
        color: #333;
	}

	.row-item{
		display: flex !important;
		align-items: baseline;
	}
	.diy_select_flex{
		flex: 1;
	}
	.query_select{
		flex: 1;
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
</style>

