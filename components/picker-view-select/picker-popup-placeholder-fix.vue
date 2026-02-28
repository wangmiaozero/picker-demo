<template>
	<view v-if="showPopup" class="popup-root">
		<view class="mask" @click="cancel"></view>
		<view class="sheet">
			<view class="toolbar">
				<text class="action cancel" @click="cancel">取消</text>
				<text class="title">{{ title }}</text>
				<text class="action confirm" @click="confirm">确定</text>
			</view>

			<!-- 思路三：添加虚拟占位项 -->
			<picker-view
				class="picker-view"
				:value="[tempIndexWithPlaceholder]"
				:indicator-style="indicatorStyle"
				@change="handleChange"
			>
				<picker-view-column>
					<view
						v-for="(item, index) in optionsWithPlaceholder"
						:key="index"
						class="picker-item"
					>
						<text v-if="item.__placeholder__">&nbsp;</text>
						<text v-else>{{ item[labelKey] }}</text>
					</view>
				</picker-view-column>
			</picker-view>
		</view>
	</view>
</template>

<script>
export default {
	name: 'PickerPopupPlaceholderFix',
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
			default: '请选择（虚拟占位项方案）'
		},
		itemHeightRpx: {
			type: Number,
			default: 88
		},
		placeholderCount: {
			type: Number,
			default: 2
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
		optionsWithPlaceholder() {
			const tail = Array.from({ length: this.placeholderCount }, () => ({
				__placeholder__: true,
				[this.labelKey]: ''
			}))
			return [...this.normalizedOptions, ...tail]
		},
		maxIndex() {
			return Math.max(this.normalizedOptions.length - 1, 0)
		},
		indicatorStyle() {
			return `height:${this.itemHeightRpx}rpx;`
		},
		tempIndexWithPlaceholder() {
			return this.clampIndex(this.tempIndex)
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
			const raw = Number((e.detail.value || [0])[0] || 0)
			this.tempIndex = this.clampIndex(raw)
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
	height: 520rpx;
}

.picker-item {
	height: 88rpx;
	line-height: 88rpx;
	text-align: center;
	font-size: 30rpx;
	color: #333;
}
</style>
