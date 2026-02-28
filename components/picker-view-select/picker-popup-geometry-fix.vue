<template>
	<view v-if="showPopup" class="popup-root">
		<view class="mask" @click="cancel"></view>
		<view class="sheet">
			<view class="toolbar">
				<text class="action cancel" @click="cancel">取消</text>
				<text class="title">{{ title }}</text>
				<text class="action confirm" @click="confirm">确定</text>
			</view>

			<!-- 思路四：结构占位（非数据占位）+ 动态几何校准 -->
			<picker-view
				class="picker-view"
				:style="pickerViewStyle"
				:value="[tempPickerIndex]"
				:indicator-style="indicatorStyle"
				@change="handleChange"
			>
				<picker-view-column>
					<view
						v-for="n in topSpacerCount"
						:key="'top-' + n"
						class="spacer"
						:style="spacerStyle"
					></view>
					<view
						v-for="(item, index) in normalizedOptions"
						:key="index"
						class="picker-item"
						:style="pickerItemStyle"
					>
						{{ item[labelKey] }}
					</view>
					<view
						v-for="n in bottomSpacerCount"
						:key="'bottom-' + n"
						class="spacer"
						:style="spacerStyle"
					></view>
				</picker-view-column>
			</picker-view>
		</view>
	</view>
</template>

<script>
export default {
	name: 'PickerPopupGeometryFix',
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
			default: '请选择（结构占位+几何校准）'
		},
		itemHeightRpx: {
			type: Number,
			default: 88
		},
		visibleItemCount: {
			type: Number,
			default: 5
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
		effectivePickerHeightRpx() {
			/*
			 * picker-view 使用“项索引”驱动选中，结构占位必须按整行高度构造，
			 * 否则会出现 299 选中后回填 300 这类错位。
			 */
			return this.effectiveVisibleItemCount * this.itemHeightRpx
		},
		resolvedItemHeightPx() {
			if (this.measuredItemHeightPx > 0) {
				return this.measuredItemHeightPx
			}
			return this.toPx(this.itemHeightRpx)
		},
		effectivePickerHeightPx() {
			return this.resolvedItemHeightPx * this.effectiveVisibleItemCount
		},
		topSpacerCount() {
			return (this.effectiveVisibleItemCount - 1) / 2
		},
		bottomSpacerCount() {
			return (this.effectiveVisibleItemCount - 1) / 2
		},
		pickerViewStyle() {
			return `height:${this.effectivePickerHeightPx}px;`
		},
		indicatorStyle() {
			/*
			 * 关键：不再手动设置 top，避免不同端对 top 的解释差异导致错位。
			 * 只传高度，让系统保持居中指示线。
			 */
			return `height:${this.resolvedItemHeightPx}px;`
		},
		spacerStyle() {
			return `height:${this.resolvedItemHeightPx}px;`
		},
		pickerItemStyle() {
			return `height:${this.resolvedItemHeightPx}px;line-height:${this.resolvedItemHeightPx}px;`
		},
		tempPickerIndex() {
			return this.clampIndex(this.tempIndex) + this.topSpacerCount
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
			const pickerIndex = Number((e.detail.value || [0])[0] || 0)
			const businessIndex = this.clampIndex(pickerIndex - this.topSpacerCount)
			this.tempIndex = businessIndex
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

.spacer {
	width: 100%;
}

.picker-item {
	text-align: center;
	font-size: 30rpx;
	color: #333;
}
</style>
