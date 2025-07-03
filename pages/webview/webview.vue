<template>
  <view>
    <web-view :src="url"></web-view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      url: ''
    };
  },
  onLoad(item) {
	if (item.download) {
		this.url = item.url + "#" + item.download
	} else {
		this.url = item.url
	}
  }
}
</script>

<style>
</style>
