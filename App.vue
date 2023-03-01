<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style>
	/*每个页面公共css */
	*{
		margin: 0;
		padding: 0;
	}

	html,body,uni-app,uni-page-wrapper,uni-page-body {
		width: 100%;
		height: 100%;
		font-size: 14px;
		font-family: Verdana, Arial, Helvetica, sans-serif;
	}
</style>
