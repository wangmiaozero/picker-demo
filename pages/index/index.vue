<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">{{ title }}</text>
		</view>
		<view class="nav-area">

			<navigator url="/pages/picker-demo/picker-demo" class="nav-link picker">
				<text class="nav-text">PickerView 组件演示 →</text>
			</navigator>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			title: 'Hello'
		}
	},
	onLoad() {
	
	},
	methods: {

	}
}
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.logo {
	height: 200rpx;
	width: 200rpx;
	margin-top: 200rpx;
	margin-left: auto;
	margin-right: auto;
	margin-bottom: 50rpx;
}

.text-area {
	display: flex;
	justify-content: center;
}

.title {
	font-size: 36rpx;
	color: #8f8f94;
}

.nav-area {
	margin-top: 80rpx;
}

.nav-link {
	padding: 20rpx 40rpx;
	background-color: #007aff;
	border-radius: 8rpx;
	display: inline-block;
	margin-bottom: 20rpx;
}

.nav-link.secondary {
	background-color: #34a853;
	display: block;
	text-align: center;
}

.nav-link.picker {
	background-color: #ff8f00;
	display: block;
	text-align: center;
}

.nav-text {
	font-size: 32rpx;
	color: #fff;
}
</style>
