<script>
	export default {
		globalData: {
			Cookie: '',
		},
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
	html,
	body {
		background-color: #eef4fc;
		width: 100%;
		height: 100%;
	}
	
	.ykt_bgc {
		background-color: #5096f5;
	}
	
	.line_1 {
		text-overflow: ellipsis;
		overflow: hidden;
		white-space: nowrap;
	}
	.line_2 {
		display: -webkit-box;
		overflow: hidden;
		text-overflow: ellipsis;
		-webkit-line-clamp: 2;
		-webkit-box-orient: vertical;
	}
	.line_3 {
		display: -webkit-box;
		overflow: hidden;
		text-overflow: ellipsis;
		-webkit-line-clamp: 3;
		-webkit-box-orient: vertical;
	}
	
	.flex-row {
		display: flex;
		flex-direction: row;
	}
	.flex-col {
		display: flex;
		flex-direction: column;
	}
	.flex-wrap {
		flex-wrap: wrap;
	}
	.flex-nowrap {
		flex-wrap: nowrap;
	}
	.justify-start {
		justify-content: flex-start;
	}
	.justify-end {
		justify-content: flex-end;
	}
	.justify-center {
		justify-content: center;
	}
	.justify-between {
		justify-content: space-between;
	}
	.justify-around {
		justify-content: space-around;
	}
	.justify-evenly {
		justify-content: space-evenly;
	}
	.items-start {
		align-items: flex-start;
	}
	.items-end {
		align-items: flex-end;
	}
	.items-center {
		align-items: center;
	}
	.items-baseline {
		align-items: baseline;
	}
	.items-stretch {
		align-items: stretch;
	}
	.self-start {
		align-self: flex-start;
	}
	.self-end {
		align-self: flex-end;
	}
	.self-center {
		align-self: center;
	}
	.self-baseline {
		align-self: baseline;
	}
	.self-stretch {
		align-self: stretch;
	}
	.flex-1 {
		flex: 1 1 0%;
	}
	.flex-auto {
		flex: 1 1 auto;
	}
	.grow {
		flex-grow: 1;
	}
	.grow-0 {
		flex-grow: 0;
	}
	.shrink {
		flex-shrink: 1;
	}
	.shrink-0 {
		flex-shrink: 0;
	}
</style>
