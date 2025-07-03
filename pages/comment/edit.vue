<template>
	<view class="page_comment" id="comment_edit">
		<!-- 评论区 -->
		<view class="comment_auth" v-if="query.nickname">
			对 {{ query.nickname }} 回复...
		
		</view>
		<uni-easyinput type="textarea" class="e-value" v-model="eValue" style="border: 1px solid #e5e5e5;"></uni-easyinput>
		<view class="btn_publish" @click="publish()" type="default">发表</view>
		<!-- /评论区 -->
	</view>
</template>

<script>
	import mixin from "@/libs/mixins/page.js"
	import tColorPicke from '@/components/diy/t-color-picker.vue';
	var _self;

	export default {
		mixins: [mixin],
		components: {
			't-color-picker': tColorPicke
		},
		data() {
			return {
				// 登录权限
				oauth: {
					"signIn": true,
					"user_group": []
				},
				form: {
					content: "ceshi"
				},
				color: {
					r: 255,
					g: 0,
					b: 0,
					a: 0.6
				},
				isEdit: false,
				fontColor: '#000',
				formats: {},
				readOnly: false,
				placeholder: '开始输入...',
				editorHeight: 300,
				keyboardHeight: 0,
				isIOS: false,
				content: "",
				editoValue: "",
				query: {
					nickname: ''
				}
				,eValue: ""
			}
		},
		onLoad() {
			_self = this;
		},

		methods: {
			cancel() {
				this.isEdit = false;
			},
			open() {
				this.$refs.colorPicker.open();
				this.isEdit = true;
				// uni.hideKeyboard();
			},
			hideKey() {
				uni.hideKeyboard();
			},
			async confirm(e) {
				this.isEdit = false;
				this.fontColor = await e.hex;
				this.onStatusChange({
					detail: {
						color: e.hex
					}
				});
				this.$forceUpdate();
			},
			readOnlyChange() {
				this.readOnly = !this.readOnly
			},
			onEditorReady() {
				uni.createSelectorQuery().select('#editor').context((res) => {
					console.log(res);
					this.editorCtx = res.context
				}).exec()
			},
			undo() {
				this.editorCtx.undo();
			},

			redo() {
				this.editorCtx.redo();
			},

			blur() {
				this.editorCtx.blur();
			},

			format(e) {
				// this.hideKey();
				let {
					name,
					value
				} = e.target.dataset;
				if (!name) return; // console.log('format', name, value)
				this.editorCtx.format(name, value);
			},

			onStatusChange(e) {
				this.formats = e.detail;
			},

			insertDivider() {
				this.editorCtx.insertDivider({
					success: function() {
						console.log('insert divider success');
					}
				});
			},

			store(e) {
				this.editorCtx.getContents({
					success: function(res) {
						e.currentTarget.id == 1 ? console.log('保存内容:', res.html) : uni.navigateTo({
							url: `../preview/preview?rich=${encodeURIComponent(res.html)}`
						});
					}
				});
			},

			clear_() {
				this.editorCtx.clear({
					success: function(res) {
						console.log("clear success");
					}
				});
			},

			removeFormat() {
				this.editorCtx.removeFormat();
			},

			insertDate() {
				const date = new Date();
				const formatDate = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
				this.editorCtx.insertText({
					text: formatDate
				});
			},

			insertImage() {
				const that = this;
				uni.chooseImage({
					count: 1,
					success: function(res) {
						that.blobToDataURI(res.tempFiles[0], function(result) {
							_self.editorCtx.insertImage({
								src: result,
								data: {
									id: 'abcd',
									role: 'god'
								},
								width: '80%',
								success: function() {
									console.log('insert image success');
								}
							});
						})
					}
				});
			},
			blobToDataURI(blob, callback) {
				console.log(blob);
				let reader = new FileReader();
				reader.readAsDataURL(blob);
				reader.onload = function(e) {
					callback(e.target.result);
				}
			},
			// 发送按钮
			async publish() {
				var content = this.eValue;
				var obj_url = getCurrentPages()[getCurrentPages().length - 1]
				var body = Object.assign({}, obj_url.options, {
					user_id: this.$store.state.user.user_id,
					nickname: this.$store.state.user.nickname,
					avatar: this.$store.state.user.avatar,
					content
				})
				
				var sensitiveWords = await this.filterSensitiveWords(body.content)
				if(sensitiveWords.length > 0){
					this.$toast('当前消息中包含敏感词: ' + sensitiveWords.join(','))
					return
				}
						
				this.$post('~/api/comment/add?', body, (res) => {
					if (res.result) {
						this.$toast(res.result, 'success');
						console.log(res.result, 'success');
						var url = this.$redirect();
						this.$nav(url);
					} else if (res.error) {
						console.log(res.error.message, 'error');
						this.$toast(res.error.message, 'error');
					}
				})
				//插入一条留言
				// this.$post('~/api/message/add?', body, (res) => {
				// 	if (res.result) {
				// 		this.$toast(res.result, 'success');
				// 		var url = this.$redirect();
				// 		this.$nav(url);
				// 	} else if (res.error) {
				// 		this.$toast(res.error.message, 'error');
				// 	}
				// })
			}
		}
	}
</script>

<style scoped>
	@import "/static/css/editor.css";

	#editor {
		background-color: #fff;
	}

	.comment_auth {
		padding-left: 10px;
	}

	.btn_publish {
		padding: 0.5rem 0.25rem;
		font-weight: 600;
		font-size: 1.25rem;
		border-radius: 2rem;
		background-color: var(--color_primary);
		color: #fff;
		text-align: center;
		margin: 0 1rem;
		margin-top: 1rem;
	}
	.e-value {
		margin-top: 30px;
		width: 80%;
		margin-left: 10%;
	}
</style>
