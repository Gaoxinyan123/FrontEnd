<template>
	<div class="page_user my_home" id="user_address">
		<div class="warp">
			<div class="container">
				<div class="row justify-content-between">
					<div class="col-12 col-md-3">
						<div class="card_menu">
							<!-- 左侧边栏 -->
						</div>
					</div>
					<div class="col-12 col-md-9">
						<div class="card_addres pl-2">
							<div class="warp">
								<div class="container-fluid">
									<div class="d-row-line">
										<div style="text-align: center;">数据统计中心</div>
									</div>
									<div class="d-row" style="height: 300px;">
																																																																																																																	</div>
								</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	import mixin from "@/libs/mixins/page.js";
	export default {
		mixins: [mixin],
		data() {
			return {
					activeName: "third",
																																																																																												url_user_count: "~/api/user/count?",
														url_article_hits: "~/api/article/sum?field=hits",
														};
		},
		mounted() {
		},
		methods: {
			async get_nickname(list,flag){
				if (flag) {
					for (let i=0;i<list.length;i++){
						await this.$get(
								"~/api/user/get_obj?user_id="+list[i],
								null,
								(json) => {
									if (json.result) {
										list[i] = json.result.obj.nickname;
									}
								});
					}
				}else {
					for (let i=0;i<list.length;i++){
						await this.$get(
								"~/api/user/get_obj?user_id="+list[i].name,
								null,
								(json) => {
									if (json.result) {
										list[i].name = json.result.obj.nickname;
									}
								});
					}
				}
			},
			// 获取饼状图统计图数据
																																																					}
		,created() {
																																								}
		,components: {
		},
	};
</script>

<style scoped>
	.container {
		min-height: 800px;
		box-shadow: none;
	}
	h5.title {
		margin: 10px 0px;
	}
</style>
