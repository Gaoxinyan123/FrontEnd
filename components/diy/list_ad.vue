<template>
	<view class="list_ad">
		<rich-text class="ad" v-for="(o, i) in list" v-if="o[vm.location] === location" :key="i" :nodes="$setRichTextImage(o[vm.content])"  @click="openUrl(o[vm.url])"></rich-text>
	</view>
</template>

<script>
	export default {
		props: {
			list: {
				type: Array,
				default: function() {
					return [];
				}
			},
			location: {
				type: String,
				default: "顶部广告"
			},
			vm: {
				type: Object,
				default: function() {
					return {
						location: "location",
						content: "content",
            url: "url"
					}
				}
			}
		},
    methods: {
      openUrl(url) {
        uni.navigateTo({
          url: `/pages/webview/webview?url=${url}`
        })
      }
    }
	}
</script>

<style>
	.list_ad{
		background-color: #fff;
	}
</style>
