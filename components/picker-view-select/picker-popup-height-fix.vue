<template>
	<view v-if="showPopup" class="popup-root">
		<view class="mask" @click="cancel"></view>
		<view class="sheet">
			<view class="toolbar">
				<text class="action cancel" @click="cancel">取消</text>
				<text class="title">{{ title }}</text>
				<text class="action confirm" @click="confirm">确定</text>
			</view>

			<!-- 思路一：增加容器高度 -->
			<picker-view
				class="picker-view"
				:style="pickerViewStyle"
				:value="[tempIndex]"
				:indicator-style="indicatorStyle"
				@change="handleChange"
			>
				<picker-view-column :style="columnStyle">
					<view
						v-for="(item, index) in normalizedOptions"
						:key="index"
						class="picker-item"
					>
						{{ item[labelKey] }}
					</view>
				</picker-view-column>
			</picker-view>
		</view>
	</view>
</template>

<script>
export default {
	name: 'PickerPopupHeightFix',
	props: {
		options: {
			type: Array,
			default() {
				return []
			}
		},
		value: {
			type: Number,
			default: 0
		},
		labelKey: {
			type: String,
			default: 'label'
		},
		visible: {
			type: Boolean,
			default: false
		},
		title: {
			type: String,
			default: '请选择（容器增高方案）'
		},
		itemHeightRpx: {
			type: Number,
			default: 88
		},
		visibleItemCount: {
			type: Number,
			default: 7
		},
		fixMode: {
			type: String,
			default: 'strict'
		}
	},
	data() {
		return {
			showPopup: false,
			innerIndex: 0,
			tempIndex: 0
		}
	},
	computed: {
		normalizedOptions() {
			return this.options.map((item) => {
				if (typeof item === 'object' && item !== null) {
					return item
				}
				return { [this.labelKey]: String(item) }
			})
		},
		maxIndex() {
			return Math.max(this.normalizedOptions.length - 1, 0)
		},
		pickerHeightRpx() {
			const safeCount = this.visibleItemCount < 3 ? 3 : this.visibleItemCount
			const oddCount = safeCount % 2 === 0 ? safeCount + 1 : safeCount
			return oddCount * this.itemHeightRpx
		},
		edgePaddingRpx() {
			return (this.pickerHeightRpx - this.itemHeightRpx) / 2
		},
		pickerViewStyle() {
			return `height:${this.pickerHeightRpx}rpx;`
		},
		columnStyle() {
			if (this.fixMode !== 'enhanced') {
				return ''
			}
			/*
			 * 增高容器后，为列增加上下缓冲区，确保首尾项都能进入选中线。
			 * 这是几何补偿，不引入任何虚拟数据项。
			 */
			return `padding-top:${this.edgePaddingRpx}rpx;padding-bottom:${this.edgePaddingRpx}rpx;box-sizing:border-box;`
		},
		indicatorStyle() {
			return `height:${this.itemHeightRpx}rpx;`
		}
	},
	watch: {
		value: {
			immediate: true,
			handler(val) {
				this.innerIndex = this.clampIndex(val)
			}
		},
		visible: {
			immediate: true,
			handler(val) {
				this.showPopup = !!val
				if (val) {
					this.tempIndex = this.clampIndex(this.value)
				}
			}
		},
		options() {
			this.innerIndex = this.clampIndex(this.innerIndex)
		}
	},
	methods: {
		clampIndex(index) {
			const parsed = Number(index)
			if (Number.isNaN(parsed)) return 0
			return Math.min(Math.max(parsed, 0), this.maxIndex)
		},
		handleChange(e) {
			this.tempIndex = this.clampIndex((e.detail.value || [0])[0])
		},
		cancel() {
			this.showPopup = false
			this.tempIndex = this.innerIndex
			this.$emit('update:visible', false)
			this.$emit('cancel')
		},
		confirm() {
			const nextIndex = this.clampIndex(this.tempIndex)
			this.innerIndex = nextIndex
			this.showPopup = false
			const payload = {
				index: nextIndex,
				item: this.normalizedOptions[nextIndex]
			}
			this.$emit('update:visible', false)
			this.$emit('input', nextIndex)
			this.$emit('change', payload)
			this.$emit('confirm', payload)
		}
	}
}
</script>

<style scoped>
.popup-root {
	position: fixed;
	left: 0;
	top: 0;
	right: 0;
	bottom: 0;
	z-index: 999;
}

.mask {
	position: absolute;
	inset: 0;
	background: rgba(0, 0, 0, 0.35);
}

.sheet {
	position: absolute;
	left: 0;
	right: 0;
	bottom: 0;
	background: #fff;
	border-radius: 24rpx 24rpx 0 0;
	overflow: hidden;
}

.toolbar {
	height: 88rpx;
	padding: 0 24rpx;
	display: flex;
	align-items: center;
	justify-content: space-between;
	border-bottom: 1rpx solid #f0f0f0;
}

.action {
	font-size: 30rpx;
}

.action.cancel {
	color: #666;
}

.action.confirm {
	color: #007aff;
}

.title {
	font-size: 30rpx;
	color: #333;
}

.picker-view {
	width: 100%;
}

.picker-item {
	height: 88rpx;
	line-height: 88rpx;
	text-align: center;
	font-size: 30rpx;
	color: #333;
}
</style>
