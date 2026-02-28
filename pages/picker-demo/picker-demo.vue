<template>
	<view class="page">
		<view class="title">Picker 最后一行不可选修复示例</view>
		<view class="desc">300 条模拟数据，四种修复策略对比：容器增高 / 指示器偏移 / 虚拟占位项 / 结构占位+几何校准</view>
		<view class="mode-row">
			<view
				class="mode-btn"
				:class="{ active: compareMode === 'strict' }"
				@click="compareMode = 'strict'"
			>
				纯思路（strict）
			</view>
			<view
				class="mode-btn"
				:class="{ active: compareMode === 'enhanced' }"
				@click="compareMode = 'enhanced'"
			>
				工程增强（enhanced）
			</view>
		</view>

		<view class="section-title">思路一：增加容器高度</view>
		<view class="picker-trigger" @click="openPicker('height')">
			<text class="picker-trigger-label">{{ heightLabel }}</text>
			<text class="picker-trigger-arrow">></text>
		</view>

		<view class="section-title">思路二：调整指示器位置</view>
		<view class="picker-trigger" @click="openPicker('indicator')">
			<text class="picker-trigger-label">{{ indicatorLabel }}</text>
			<text class="picker-trigger-arrow">></text>
		</view>

		<view class="section-title">思路三：添加虚拟占位项（推荐）</view>
		<view class="picker-trigger" @click="openPicker('placeholder')">
			<text class="picker-trigger-label">{{ placeholderLabel }}</text>
			<text class="picker-trigger-arrow">></text>
		</view>

		<view class="section-title">思路四：结构占位 + 动态几何校准（更优解）</view>
		<view class="picker-trigger" @click="openPicker('geometry')">
			<text class="picker-trigger-label">{{ geometryLabel }}</text>
			<text class="picker-trigger-arrow">></text>
		</view>

		<picker-popup-height-fix
			:options="options"
			v-model="selectedHeightIndex"
			:visible="showHeightPicker"
			title="请选择（容器增高）"
			:fix-mode="compareMode"
			@update:visible="showHeightPicker = $event"
		/>

		<picker-popup-indicator-fix
			:options="options"
			v-model="selectedIndicatorIndex"
			:visible="showIndicatorPicker"
			title="请选择（指示器偏移）"
			:indicator-offset-rpx="24"
			:fix-mode="compareMode"
			@update:visible="showIndicatorPicker = $event"
		/>

		<picker-popup-placeholder-fix
			:options="options"
			v-model="selectedPlaceholderIndex"
			:visible="showPlaceholderPicker"
			title="请选择（虚拟占位项）"
			:placeholder-count="2"
			@update:visible="showPlaceholderPicker = $event"
		/>

		<picker-popup-geometry-fix
			:options="options"
			v-model="selectedGeometryIndex"
			:visible="showGeometryPicker"
			title="请选择（结构占位+几何校准）"
			@update:visible="showGeometryPicker = $event"
		/>

		<view class="panel compare">
			<text class="panel-text">总条数：{{ options.length }}</text>
			<text class="panel-text">思路一索引：{{ selectedHeightIndex }}，值：{{ heightLabel }}</text>
			<text class="panel-text">思路二索引：{{ selectedIndicatorIndex }}，值：{{ indicatorLabel }}</text>
			<text class="panel-text">思路三索引：{{ selectedPlaceholderIndex }}，值：{{ placeholderLabel }}</text>
			<text class="panel-text">思路四索引：{{ selectedGeometryIndex }}，值：{{ geometryLabel }}</text>
			<text class="panel-tip">当前模式：{{ compareMode === 'strict' ? '纯思路（不叠加补偿）' : '工程增强（叠加边界补偿）' }}</text>
			<text class="panel-tip">建议重点验证第 299/300 条。思路四在“数据纯净 + 稳定性”之间通常更均衡。</text>
		</view>
	</view>
</template>

<script>
import PickerPopupHeightFix from '@/components/picker-view-select/picker-popup-height-fix.vue'
import PickerPopupIndicatorFix from '@/components/picker-view-select/picker-popup-indicator-fix.vue'
import PickerPopupPlaceholderFix from '@/components/picker-view-select/picker-popup-placeholder-fix.vue'
import PickerPopupGeometryFix from '@/components/picker-view-select/picker-popup-geometry-fix.vue'

export default {
	components: {
		PickerPopupHeightFix,
		PickerPopupIndicatorFix,
		PickerPopupPlaceholderFix,
		PickerPopupGeometryFix
	},
	data() {
		return {
			options: [],
			selectedHeightIndex: 0,
			selectedIndicatorIndex: 0,
			selectedPlaceholderIndex: 0,
			selectedGeometryIndex: 0,
			showHeightPicker: false,
			showIndicatorPicker: false,
			showPlaceholderPicker: false,
			showGeometryPicker: false,
			compareMode: 'strict'
		}
	},
	computed: {
		heightLabel() {
			const current = this.options[this.selectedHeightIndex]
			return current ? current.label : '-'
		},
		indicatorLabel() {
			const current = this.options[this.selectedIndicatorIndex]
			return current ? current.label : '-'
		},
		placeholderLabel() {
			const current = this.options[this.selectedPlaceholderIndex]
			return current ? current.label : '-'
		},
		geometryLabel() {
			const current = this.options[this.selectedGeometryIndex]
			return current ? current.label : '-'
		}
	},
	onLoad() {
		this.options = Array.from({ length: 300 }, (_, i) => {
			const order = i + 1
			return {
				label: `第 ${order} 条数据`,
				value: `item-${order}`
			}
		})
	},
	methods: {
		openPicker(type) {
			if (type === 'height') this.showHeightPicker = true
			if (type === 'indicator') this.showIndicatorPicker = true
			if (type === 'placeholder') this.showPlaceholderPicker = true
			if (type === 'geometry') this.showGeometryPicker = true
		}
	}
}
</script>

<style scoped>
.page {
	padding: 28rpx;
	background: #f5f6f7;
	min-height: 100vh;
	box-sizing: border-box;
}

.title {
	font-size: 38rpx;
	font-weight: 600;
	color: #222;
	margin-bottom: 12rpx;
}

.desc {
	font-size: 28rpx;
	color: #666;
	margin-bottom: 20rpx;
	line-height: 1.6;
}

.section-title {
	font-size: 28rpx;
	color: #444;
	margin-top: 20rpx;
	margin-bottom: 12rpx;
}

.mode-row {
	display: flex;
	gap: 16rpx;
	margin-bottom: 12rpx;
}

.mode-btn {
	flex: 1;
	height: 72rpx;
	border-radius: 10rpx;
	background: #fff;
	color: #666;
	font-size: 26rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	border: 1rpx solid #e9e9e9;
}

.mode-btn.active {
	color: #fff;
	background: #007aff;
	border-color: #007aff;
}

.picker-trigger {
	height: 88rpx;
	padding: 0 24rpx;
	background: #fff;
	border-radius: 12rpx;
	display: flex;
	align-items: center;
	justify-content: space-between;
}

.picker-trigger-label {
	font-size: 30rpx;
	color: #333;
}

.picker-trigger-arrow {
	font-size: 28rpx;
	color: #999;
}

.panel {
	margin-top: 24rpx;
	padding: 24rpx;
	background: #fff;
	border-radius: 12rpx;
}

.panel.compare {
	margin-top: 32rpx;
}

.panel-text {
	display: block;
	font-size: 28rpx;
	color: #333;
	line-height: 1.8;
}

.panel-tip {
	display: block;
	margin-top: 8rpx;
	font-size: 24rpx;
	color: #007aff;
}
</style>
