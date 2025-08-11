<template>
	<view>
		<view class="confirm-content cursor" @click="showModal">
			<text class="ellipsis">{{title === '' ? placeholder : title}}</text>
			<text @click.stop="empty" class="cursor">x</text>
		</view>
		<view class="cascader-modal" :class="{show:isShowModal}" @click="hideModal">
			<view class="cascader-dialog" @click.stop="">
				<view class="cascader-title">
					<text class="cursor" @click.stop="hideModal('hide')">关闭</text>
					<view class="title">请选择</view>
					<text class="cursor" @click.stop="hideModal">确认</text>
				</view>
				<view class="cascader-tags">
					<view @click.stop="switchDate(index)" class="item cursor"
						:class="{'sub-active':index === selectIndex}" v-for="(item,index) in currentTitles"
						:key="index">
						{{item}}
					</view>
				</view>
				<view class="input-box">
					<input placeholder="查询" ref="inputRef" :value="value" @input="change">
				</view>
				<view class="cascader-content" @click.stop="">
					<view v-for="(item,index) in currentList" :key="index" class="cursor"
						:class="{'active-content':judeg(item.index,item.data[valueKey])}" @click.stop="select(item)">
						<view class="title">{{item.data[labelKey]}}</view>
						<text v-if="judeg(item.index,item.data[valueKey])">✔</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			placeholder: {
				default: "请选择",
				type: String
			},
			list: {
				type: Array,
				default: () => [],
			},
			valueKey: {
				default: 'id',
				type: String
			},
			labelKey: {
				default: 'label',
				type: String
			},
			values: {
				type: Array,
				default: () => [],
			}
		},
		data() {
			return {
				value: '',
				isShowModal: false,
				selectIndex: 0,
				currentList: [],
				currentTitles: [],
				currentData: [],
				saveList: [],
				title: '',
				setValue: null,
				depth: 0,
				timer: null,
			}
		},
		watch: {
			list: {
				async handler(val) {
					// tree 数据执行
					if (this.values && this.values.length) {
						this.setValue = new Set(this.values)
						this.currentList = await this.dataDisplay(this.list)
						this.setTitle()
						this.selectIndex = this.values.length - 1
					}
					// 懒加载执行
					else {
						this.currentList = this.setIndex(this.list)
					}
				},
				immediate: true
			}
		},
		methods: {
			judeg(index, item) {
				return this.currentData[index] && this.currentData[index][this.valueKey] === item
			},

			/**
			 * 数据回显 
			 **/
			dataDisplay(data) {
				let item = null
				let num = null

				let f = (data, resolve) => {
					for (item of data) {
						if (this.setValue.has(item[this.valueKey])) {
							num = this.depth - 1 === -1 ? null : this.depth - 1
							this.currentData.push(item)
							this.currentTitles.push(item[this.labelKey])
							this.depth++
							if (this.setValue.size === this.currentTitles.length) {
								return resolve(this.setIndex(data, num))
							} else {
								this.setIndex(data, num) // 保存数据到 saveList
							}
						}
						if (item.children && item.children.length) {
							f(item.children, resolve)
						}
					}
				}

				return new Promise(resolve => {
					f(data, resolve)
				})
			},

			/**
			 * 设置 index 字段，index 表示层级 
			 **/
			setIndex(data, index = null) {
				let item = null
				let num = 0
				this.saveList[index === null ? 0 : index + 1] = data.map(item => {
					num = index === null ? 0 : index + 1
					return this.setObj(item, num)
				})

				return this.saveList[index === null ? 0 : index + 1]
			},

			setObj(item, num) {
				return {
					data: item,
					index: num
				}
			},

			/**
			 * 懒加载
			 **/
			load(node, children) {
				if (children && children.length) {
					let data = this.setIndex(children, node.index)
					this.currentList = data
				}
			},

			/**
			 * 点击选项
			 **/
			select(item) {
				// 点击父级，删除选中的子节点
				if (this.currentData.length && item.index < this.currentData.length - 1 &&
					this.currentData[item.index][this.valueKey] !== item.data[this.valueKey]) {
					this.saveList.splice(item.index + 1, this.currentData.length - 1)
					this.currentTitles.splice(item.index + 1, this.currentData.length - 1)
					this.currentData.splice(item.index + 1, this.currentData.length - 1)
				}

				// tree 数据执行
				if (item.data.children && item.data.children.length) {
					this.currentList = this.setIndex(item.data.children, item.index)
				}

				this.$set(this.currentData, item.index, item.data)
				this.$set(this.currentTitles, item.index, item.data[this.labelKey])
				this.selectIndex = item.index
				this.$emit('change', this.currentData.map(item => item[this.valueKey]), item.data, item)
			},

			/**
			 * 切换数据 
			 **/
			switchDate(index) {
				this.currentList = this.saveList[index]
				this.selectIndex = index
			},

			/**
			 * 清空 
			 **/
			empty() {
				this.currentList = this.saveList[0]
				this.currentData = []
				this.currentTitles = []
				this.saveList = [this.saveList[0]]
				this.selectIndex = 0
				this.setTitle()
			},

			/**
			 * 显示 
			 **/
			showModal() {
				if (!this.disabled) {
					this.isShowModal = true
					this.$emit('visible-change', true)
				}
			},

			/**
			 * 隐藏 
			 **/
			hideModal(key) {
				this.isShowModal = false
				this.$emit('visible-change', false)
				if (key !== 'hide') {
					this.setTitle()
					this.$emit("confirm", this.currentData.map(item => item[this.valueKey]))
				}
			},

			/**
			 * 展示选中的节点 
			 **/
			setTitle() {
				this.title = this.currentTitles.length ? this.currentTitles.join('/') : this.placeholder
			},

			change(val) {
				if (this.timer) {
					clearTimeout(this.timer)
				}
				if (val.detail.value === '') {
					this.currentList = this.saveList[this.selectIndex]
					return;
				}
				let item = null
				this.timer = setTimeout(() => {
					let arr = []
					for (item of this.currentList) {
						if (item.data[this.labelKey].indexOf(val.detail.value) > -1) {
							arr.push(item)
						}
					}
					this.currentList = arr
				}, 200)
			}
		}
	}
</script>
<style lang="scss" scoped>
	.ellipsis {
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
	}

	.input-box {
		border: 1px solid #ccc
	}

	.sub-active {
		position: relative;

		&:before {
			content: '';
			position: absolute;
			bottom: 0;
			left: 0;
			display: block;
			height: 2px;
			width: 100%;
			background-color: cornflowerblue;
		}
	}

	.sub-active text {
		color: cornflowerblue;
	}

	.active-content {
		color: cornflowerblue;
	}

	.cursor {
		cursor: pointer;
	}

	.cascader-content>view {
		padding: 15rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.confirm-content {
		display: flex;
		align-items: center;
		padding: 20rpx;
		border-radius: 10rpx;
		border: rgba(0, 0, 0, .5) solid 1rpx;

	}

	.confirm-content text:last-child {
		margin-left: 15rpx;
	}

	.cascader-content {
		text-align: left;
		margin: 20rpx;
		background-color: #FFFFFF;
		height: 600rpx;
		overflow: auto;
	}

	.cascader-modal {
		position: fixed;
		bottom: 0;
		left: 0;
		z-index: 9999;
		visibility: hidden;
		height: 100%;
		width: 100%;
		background: rgba(0, 0, 0, 0.6);
	}

	.cascader-modal.show {
		visibility: visible;
	}

	.cascader-dialog {
		overflow: hidden;
		background-color: #fff;
		position: absolute;
		bottom: -2000px;
		left: 0;
		width: 100%;
		border-top-left-radius: 30rpx;
		border-top-right-radius: 30rpx;
		transition: bottom 0.5s;
	}

	.show .cascader-dialog {
		bottom: 0;
	}

	.cascader-tags {
		overflow: auto;
		display: flex;
		white-space: nowrap;
		padding: 20rpx 20rpx 0 20rpx;
		height: 50rpx;
		border: 1rpx solid #e8eaed;
	}

	.cascader-tags .item {
		padding-left: 10rpx;
		padding-right: 30rpx;
		padding-bottom: 10rpx;
	}

	.cascader-title {
		padding: 0 20rpx;
		display: flex;
		position: relative;
		align-items: center;
		height: 100rpx;
		justify-content: space-between;
	}
</style>