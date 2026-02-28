<template>
	<view v-if="showPopup" class="popup-root">
		<view class="mask" @click="cancel"></view>
		<view class="sheet">
			<view class="toolbar">
				<text class="action cancel" @click="cancel">取消</text>
				<text class="title">{{ title }}</text>
				<text class="action confirm" @click="confirm">确定</text>
			</view>

			<!-- 思路二：调整指示器位置 -->
			<picker-view
				class="picker-view indicator-fix"
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
						:style="pickerItemStyle"
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
	name: 'PickerPopupIndicatorFix',
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
			default: '请选择（指示器偏移方案）'
		},
		itemHeightRpx: {
			type: Number,
			default: 88
		},
		pickerHeightRpx: {
			type: Number,
			default: 520
		},
		indicatorOffsetRpx: {
			type: Number,
			default: 24
		},
		visibleItemCount: {
			type: Number,
			default: 5
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
			tempIndex: 0,
			measuredItemHeightPx: 0
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
		effectiveVisibleItemCount() {
			const safe = this.visibleItemCount < 3 ? 3 : this.visibleItemCount
			return safe % 2 === 0 ? safe + 1 : safe
		},
		baseItemHeightPx() {
			return Math.round(this.toPx(this.itemHeightRpx))
		},
		resolvedItemHeightPx() {
			if (this.measuredItemHeightPx > 0) {
				return this.measuredItemHeightPx
			}
			return this.baseItemHeightPx
		},
		effectivePickerHeightPx() {
			/*
			 * 使用整像素高度，避免出现 274.5px 这类半像素 padding 导致的视觉错位。
			 */
			return this.resolvedItemHeightPx * this.effectiveVisibleItemCount
		},
		pickerViewStyle() {
			return `height:${this.effectivePickerHeightPx}px;`
		},
		indicatorOffsetPx() {
			return Math.round(this.toPx(this.indicatorOffsetRpx))
		},
		alignedIndicatorOffsetPx() {
			/*
			 * 关键：思路二如果偏移量不是“行高的整数倍”，选中线就会落在两行中间。
			 * 这里做吸附，保证指示器始终和 item 网格对齐。
			 */
			if (!this.resolvedItemHeightPx) return 0
			return Math.round(this.indicatorOffsetPx / this.resolvedItemHeightPx) * this.resolvedItemHeightPx
		},
		indicatorTopPx() {
			const centerTop = (this.effectivePickerHeightPx - this.resolvedItemHeightPx) / 2
			const next = centerTop + this.alignedIndicatorOffsetPx
			const maxTop = this.effectivePickerHeightPx - this.resolvedItemHeightPx
			return Math.min(Math.max(next, 0), maxTop)
		},
		columnStyle() {
			if (this.fixMode !== 'enhanced') {
				return ''
			}
			const top = Math.round(this.indicatorTopPx)
			const bottom = Math.round(this.effectivePickerHeightPx - this.indicatorTopPx - this.resolvedItemHeightPx)
			/*
			 * 指示器偏移后，同步补偿列上下滚动缓冲区，确保首尾项仍可精确到达选中线。
			 */
			return `padding-top:${top}px;padding-bottom:${bottom}px;box-sizing:border-box;`
		},
		indicatorStyle() {
			if (this.fixMode === 'enhanced') {
				return `height:${this.resolvedItemHeightPx}px;top:${Math.round(this.indicatorTopPx)}px;`
			}
			/*
			 * strict: 仅调整指示器偏移，不叠加上下缓冲补偿。
			 */
			return `height:${this.resolvedItemHeightPx}px;margin-top:${this.alignedIndicatorOffsetPx}px;`
		},
		pickerItemStyle() {
			return `height:${this.resolvedItemHeightPx}px;line-height:${this.resolvedItemHeightPx}px;`
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
					this.measuredItemHeightPx = 0
					this.$nextTick(() => {
						this.measureItemHeight()
					})
				}
			}
		},
		options() {
			this.innerIndex = this.clampIndex(this.innerIndex)
		}
	},
	methods: {
		toPx(upx) {
			if (typeof uni !== 'undefined' && typeof uni.upx2px === 'function') {
				return uni.upx2px(upx)
			}
			return upx / 2
		},
		clampIndex(index) {
			const parsed = Number(index)
			if (Number.isNaN(parsed)) return 0
			return Math.min(Math.max(parsed, 0), this.maxIndex)
		},
		measureItemHeight() {
			if (!this.showPopup) return
			const query = uni.createSelectorQuery().in(this)
			query.select('.picker-item').boundingClientRect((rect) => {
				if (!rect || !rect.height) return
				const next = Math.round(rect.height)
				if (next > 0 && next !== this.measuredItemHeightPx) {
					this.measuredItemHeightPx = next
				}
			}).exec()
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
	text-align: center;
	font-size: 30rpx;
	color: #333;
}
</style>
