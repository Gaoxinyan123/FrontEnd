<template>
  <view>
			<div id ="outerdiv" :style="style_outer">
				<web-view id ='inneriframe' src="http://m.qsbdc.com/learn_word/" :style="style_inner"></web-view>
			</div>
  </view>
</template>

<script lang="renderjs">
export default {
  data() {
    return {
		style_outer:{},
		style_inner:{}
    };
  },
  onLoad() {
	  this.style_outer = {'height':(document.body.clientHeight)+'px'}
	  this.style_inner = {'height':(document.body.clientHeight-(-50))+'px'}
  }
}
</script>

<style>
	#outerdiv
	    {
	      margin: 0 auto;
	      width:100%;
	      overflow:hidden;
	      position:relative;
	    }
	
	#inneriframe
	{
	  position:absolute;
	  top:-50px;
	  left:0px;
	  width:100%;
	}
</style>
