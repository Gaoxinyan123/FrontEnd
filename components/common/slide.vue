<template>
	<view class="swiper_img">
		<swiper class="swiper" :indicator-color="indicatorColor" :indicator-active-color="indicatorActiveColor"
			:indicator-dots="indicatorDots" :autoplay="autoplay" :interval="interval" :duration="duration">
			<swiper-item v-for="(o, i) in list" :key="i">
				<image v-if="o[vm.img]" :src="$fullImgUrl(o[vm.img])"
					@click="previewImage(i)"
					@contextmenu.prevent="onContextMenu($fullImgUrl(o[vm.img]), $event)"
				></image>
				<text class="title" v-if="show_title && o[vm.title]">{{ o[vm.title] }}</text>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
	export default {
        name: "SwiperImg",
		props: {
			show_title: {
				type: String,
				default: ""
			},
			list: {
				type: Array,
				default: function() {
					return [];
				}
			},
			vm: {
				type: Object,
				default: function() {
					return {
						img: "img",
						title: "title"
					}
				}
			}
		},
		data() {
			return {
				background: ['color1', 'color2', 'color3'],
				indicatorDots: true,
				indicatorColor: "rgba(0, 0, 0, .3)",
				indicatorActiveColor: "#fff",
				autoplay: true,
				interval: 2000,
				duration: 500
			}
		},
		methods: {
			changeIndicatorDots(e) {
				this.indicatorDots = !this.indicatorDots
			},
			changeAutoplay(e) {
				this.autoplay = !this.autoplay
			},
			intervalChange(e) {
				this.interval = e.target.value
			},
			durationChange(e) {
				this.duration = e.target.value
			},
			saveImage(url) {
				// #ifdef H5
				// H5 右键保存即可，无需处理
				// #endif
				// #ifdef MP-WEIXIN
				uni.saveImageToPhotosAlbum({
					filePath: url,
					success: function () {
						uni.showToast({ title: '图片已保存', icon: 'success' });
					},
					fail: function () {
						uni.showToast({ title: '保存失败', icon: 'none' });
					}
				});
				// #endif
			},
			onContextMenu(url, event) {
				// H5下可弹出自定义菜单或直接提示用户右键保存
				// 这里只做简单提示
				uni.showToast({ title: '请右键图片选择"图片另存为"', icon: 'none' });
			},
			previewImage(index) {
				// 预览所有图片，定位到当前点击的那一张
				const urls = this.list.map(o => this.$fullImgUrl(o[this.vm.img]));
				uni.previewImage({
					urls: urls,
					current: urls[index],
					success: () => {},
					fail: () => {}
				});
			}
		}
	}
</script>

<style lang="scss" scoped>
@media (min-width:768px) {
    .swiper_img .swiper {
        height: 250px;
    }
}

.swiper_img {
    image {
        width: 100%;
        height: 100%;
    }
    .title {
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        text-align: center;
    }
}
</style>
