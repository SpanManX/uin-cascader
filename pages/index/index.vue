<template>
	<view class="content">
		<cascader :values="value" :list="tree" @confirm="confirm" valueKey="value" placeholder="点击选择设备"></cascader>
		<cascader ref="cascader" :list="options" @change="change" @confirm="confirm" valueKey="value"
			placeholder="点击选择设备(懒加载)">
		</cascader>
	</view>
</template>

<script>
	import cascader from './cascader.vue'
	export default {
		components: {
			cascader
		},
		data() {
			return {
				value: ['2', '2-2', '2-3-1', '2-3-1-2'],
				title: 'Hello',
				options: [{
					label: '一级-1',
					value: '1'
				}, {
					label: '一级-2',
					value: '2'
				}, {
					label: '一级-3',
					value: '3'
				}, {
					label: '一级-4',
					value: '4'
				}, {
					label: '一级-5',
					value: '5'
				}, {
					label: '一级-6',
					value: '6'
				}, {
					label: '一级-7',
					value: '7'
				}, {
					label: '一级-8',
					value: '8'
				}, {
					label: '一级-9',
					value: '9'
				}, {
					label: '一级-10',
					value: '10'
				}, {
					label: '一级-11',
					value: '11'
				}],
				tree: [{
						value: '1',
						label: 'Level one 1',
						children: [{
							value: '1-1',
							label: 'Level two 1-1',
							children: [{
								value: '1-1-1',
								label: 'Level three 1-1-1',
							}, ],
						}, ],
					},
					{
						value: '2',
						label: 'Level one 2',
						children: [{
								value: '2-1',
								label: 'Level two 2-1',
								children: [{
									value: '2-1-1',
									label: 'Level three 2-1-1',
									children: [{
										value: '2-1-2-1',
										label: 'Level three 2-1-2-1',
									}, {
										value: '2-1-2-2',
										label: 'Level three 2-1-2-2',
									}]
								}, ],
							},
							{
								value: '2-2',
								label: 'Level two 2-2',
								children: [{
									value: '2-2-1',
									label: 'Level three 2-2-1',
									children: [{
										value: '2-2-3-1',
										label: 'Level three 2-2-3-1',
									}, {
										value: '2-2-3-2',
										label: 'Level three 2-2-3-2',
									}]
								}, {
									value: '2-3-1',
									label: 'Level three 2-3-1',
									children: [{
										value: '2-3-1-1',
										label: 'Level three 2-3-1-1',
									}, {
										value: '2-3-1-2',
										label: 'Level three 2-3-1-2',
									}]
								}, ],
							},
						],
					},
					{
						value: '3',
						label: 'Level one 3',
						children: [{
								value: '3-1',
								label: 'Level two 3-1',
								children: [{
									value: '3-1-1',
									label: 'Level three 3-1-1',
								}, ],
							},
							{
								value: '3-2',
								label: 'Level two 3-2',
								children: [{
									value: '3-2-1',
									label: 'Level three 3-2-1',
								}, ],
							},
						],
					},
				]
			}
		},
		methods: {
			change(val, item, item1) {
				if (val.length === 1) {
					this.$refs.cascader.load(item1, [{
						label: `${item.label}_二级-1`,
						value: '121212'
					}, {
						label: `${item.label}_二级-2`,
						value: '6666666666'
					}, {
						label: `${item.label}_二级-3`,
						value: '00000000'
					}])
				} else if (val.length === 2) {
					this.$refs.cascader.load(item1, [{
						label: `${item.label}_三级-1`,
						value: '434343'
					}, {
						label: `${item.label}_三级-2`,
						value: 'mmmmmmmm'
					}])
				} else {
					this.$refs.cascader.load(item1, [])
				}
			},

			confirm(data) {
				console.log(data)
			}
		}
	}
</script>

<style>
	cascader {
		width: 91%;
		margin-right: 20rpx;

		/deep/ .input {
			background-color: #fff;
		}
	}

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
</style>