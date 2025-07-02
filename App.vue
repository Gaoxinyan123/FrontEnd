<script>
	import host from "libs/plugins/port_config.js"
	export default {
		onLaunch: function() {
		},
		onShow: function() {
			console.log("App Show");
			var token = uni.db.get("token");
			this.$store.commit("user_set", {
				token: token,
			});
		},
		onHide: function() {
			console.log("App Hide");
		},
		/**
		 * vue的store刷新丢失问题，刷新页面时vue实例重新加载，store就会被重置，可以把定义刷新前把store存入本地localStorage、sessionStorage、cookie中，localStorage是永久储存，
		 * 重新打开页面时会读取上一次打开的页面数据，sessionStorage是储存到关闭为止，cookie不适合存大量数据。根据我的需求，最合适的是sessionStorage。
		 * beforeunload在页面刷新时触发，可以监听这个方法，让页面在刷新前存store到sessionStorage中。
		 * 当然，在页面刷新时还要读取sessionStorage中的数据到store中，读取和储存都写在app.vue中。
		 */
		created() {
			// 在页面加载时读取sessionStorage
			if (typeof(sessionStorage) != "undefined" && sessionStorage.getItem('store')) {
				this.$store.replaceState(Object.assign({}, this.$store.state, JSON.parse(sessionStorage.getItem('store'))))
			}
			// 在页面刷新时将store保存到sessionStorage里
			// window.addEventListener('beforeunload', () => {
			//   sessionStorage.setItem('store', JSON.stringify(this.$store.state))
			// })
		}
		,globalData: {
			host: host.slice(0, -1)
		}
	};
</script>


<style lang="scss">
	.a-start {
		display: flex;
		align-items: flex-start;
	}
	.j-between {
		display: flex;
		justify-content: space-between;
	}
	@import "./static/css/icon.css";
	@import "./static/css/common.css";
	@import "./static/css/theme.css";
	@import "./static/css/diy.scss";
</style>