<template>
	<view class="content">

		<!-- 状态栏 -->
		<view v-if="hasData" class="todo-header">
			<view class="todo-header_left">
				<text class="active-text">{{text}}</text>
				<text>{{ listData.length }}条</text>
			</view>
			<view class="todo-header_right">
				<view class="todo-header_right-item" :class="{'active-tab': activeIndex == 0}" @click="tab(0)">全部</view>
				<view class="todo-header_right-item" :class="{'active-tab': activeIndex == 1}" @click="tab(1)">代办</view>
				<view class="todo-header_right-item" :class="{'active-tab': activeIndex == 2}" @click="tab(2)">已完成
				</view>
			</view>
		</view>

		<!-- 无数据页面 -->
		<view v-if="!hasData" class="default">
			<view class="default-image">
				<image src="../../static/default.png" mode="aspectFit"></image>
			</view>
			<view class="default-text">
				<view class="default-info_text">您还没有创建任何代办事项</view>
				<view class="default-info_text">点击下方＋号创建一个吧</view>
			</view>
		</view>

		<!-- 内容 -->
		<view v-else class="todo-content">
			<view class="todo-list" :class="{'todo-finish': item.checked}" v-for="(item, index) in listData"
				:key="index" @click="handleFinish(item.id)" @touchstart="touchStart" @touchend="touchEnd">
				<view class="todo-list_checkbox">
					<view class="checkbox"></view>
				</view>
				<view class="todo-list_content">
					{{item.content}}
				</view>
				<view class="todo-list_dele" v-if="deleFinish">
					删除
				</view>
			</view>
		</view>

		<!--  创建按钮 -->
		<view class="create-todo" @click="openCreate">
			<text class="iconfont icon-jia" :class="{'create-todo-active': textShow}"></text>
		</view>

		<!-- 输入框 -->
		<view v-if="active" class="create-content" :class="{'create-show': textShow}">
			<view class="create-content-box">
				<!-- input 输入 -->
				<view class="create-input">
					<input type="text" v-model="todo" placeholder="请输入代办事项">
				</view>
				<!-- 发布按钮 -->
				<view class="craete-button" @click="handleCreate">
					创建
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		computed: {
			hasData() {
				return this.list && this.list.length != 0
			},
			listData() {
				const all = JSON.parse(JSON.stringify(this.list))
				let newList = []
				if (this.activeIndex === 0) {
					this.text = '全部'
					return all
				} else if (this.activeIndex === 1) {
					this.text = '代办'
					newList = all.filter(item => {
						return item.checked === false
					})
					return newList
				} else if (this.activeIndex === 2) {
					this.text = '已完成'
					newList = all.filter(item => {
						return item.checked === true
					})
					return newList
				}
				return []
			}
		},
		data() {
			return {
				list: [],
				todo: '',
				startTime: 0,
				startPosition: 0,
				endPosition: 0,
				active: false,
				deleFinish: false,
				activeIndex: 0,
				text: '全部',
				textShow: false
			}
		},
		onLoad() {},
		methods: {
			// 打开输入框
			openCreate() {
				const _this = this
				_this.active = !_this.active

				_this.textShow = !_this.textShow
			},
			handleCreate() {
				if (this.todo === '') {
					uni.showToast({
						title: '请输入内容',
						icon: 'none'
					})
					return
				}
				this.list.unshift({
					id: new Date().getTime(),
					content: this.todo,
					checked: false
				})
				this.todo = ''
				this.textShow = false
				this.active = false
				try {
					uni.setStorageSync('storage_key', this.list);
				} catch (e) {
					// error
				}
				uni.showToast({
					title: '发布成功',
					icon: 'none'
				})
			},
			handleFinish(id) {
				const index = this.list.findIndex(item => {
					return item.id == id
				})
				this.list[index].checked = !this.list[index].checked
			},
			// 起点
			touchStart(event) {
				this.startTime = Date.now()
				this.startPosition = event.changedTouches[0].clientX
			},
			// 终点,计算移动距离
			touchEnd(event) {
				const endTime = Date.now()
				if (endTime - this.startTime > 2000) {
					return;
				}
				this.endPosition = event.changedTouches[0].clientX


				//当移动距离超过10时判断左滑右滑。
				if (Math.abs(this.endPosition - this.startPosition) > 10) {
					this.endPosition = event.changedTouches[0].clientX
					if (this.endPosition - this.startPosition > 0) {
						this.deleFinish = false;
					} else {
						this.deleFinish = true;
					}

				} else {
					return;
				}

				console.log(elePosition)
			},
			moveFinish() {
				this.deleFinish = true;
			},
			tab(index) {
				this.activeIndex = index
			}
		}
	}
</script>

<style lang="scss" scoped>
	@import '@/common/icon.css';

	.todo-header {
		display: flex;
		position: fixed;
		width: 100%;
		height: 45px;
		padding: 0 15px;
		align-items: center;
		box-shadow: -1px 1px 5px 0 rgba(0, 0, 0, 0.1);
		background: #ffffff;
		font-size: 12px;
		color: #333333;
		z-index: 10;
		box-sizing: border-box;

		.todo-header_left {
			width: 100%;
			flex-shrink: 1;
		}

		.todo-header_right {
			flex-shrink: 0;
			display: flex;

			.todo-header_right-item {
				padding: 0 5px;
			}
		}
	}

	.active-tab {
		color: #279ABF;
	}

	.active-text {
		font-size: 14px;
		color: #279ABF;
		padding-right: 10px;
	}

	.todo-content {
		position: relative;
		padding-top: 50px;
		padding-bottom: 90px;

		.todo-list {
			display: flex;
			justify-content: space-between;
			align-items: center;
			position: relative;
			padding-left: 15px;
			margin: 15px;
			border-radius: 10px;
			background: #cfebfd;
			color: #0c3854;
			font-size: 14px;
			box-shadow: -1px 1px 5px 1px rgba(0, 0, 0, 0.1), -1px 2px 1px 0 rgba(255, 255, 255, 0) inset;
			overflow: hidden;
			line-height: 50px;

			.todo-list_checkbox {
				padding-right: 15px;

				.checkbox {
					width: 20px;
					height: 20px;
					border-radius: 50%;
					background: #ffffff;
					box-shadow: 0 0 5px 1px rgba(0, 0, 0, 0.1);
				}
			}

			.todo-list_content {
				padding-right: 20px;
			}

			.todo-list_dele {
				color: #000;
				background-color: #EE0A24;
				width: 80px;
				text-align: center;
			}
		}

		.todo-list:after {
			position: absolute;
			content: '';
			top: 0;
			left: 0;
			bottom: 0;
			width: 5px;
			background: #91d1e8;
		}
	}

	.todo-finish {
		.checkbox {
			position: relative;
			background: #eee !important;
		}

		.checkbox:after {
			position: absolute;
			content: '';
			width: 10px;
			height: 10px;
			top: 0;
			left: 0;
			right: 0;
			bottom: 0;
			margin: auto;
			border-radius: 50%;
			box-shadow: 0 0 2px 0px rgba(0, 0, 0, 0.2) inset;
			background: #cfebfd;
		}

		.todo-list_content {
			color: #999999;
		}

		.todo-list_content:after {
			content: '';
			position: absolute;
			top: 0;
			bottom: 0;
			left: 40px;
			right: 10px;
			height: 2px;
			margin: auto 0;
			background: #bdcdd8;
		}
	}

	.todo-finish.todo-list:after {
		background: #cccccc;
	}

	.create-todo {
		display: flex;
		justify-content: center;
		align-items: center;
		position: fixed;
		bottom: 20px;
		left: 0;
		right: 0;
		margin: 0 auto;
		width: 50px;
		height: 50px;
		border-radius: 50%;
		background-color: #deeff5;
		box-shadow: -1px 1px 5px 2px rgba(0, 0, 0, 0.1), -1px 1px 1px 0 rgba(255, 255, 255, 0) inset;
	}

	.icon-jia {
		font-size: 30px;
		color: #add8e6;
	}

	.create-content {
		position: fixed;
		bottom: 95px;
		left: 20px;
		right: 20px;
		transition: all 0.3s;
		opacity: 0;
		transform: scale(0) translateY(200%);

		.create-content-box {
			display: flex;
			align-items: center;
			padding: 0 0 0 15px;
			border-radius: 50px;
			background: #deeff5;
			box-shadow: -1px 1px 5px 2px rgba(0, 0, 0, 0.1), -1px 1px 1px 0 rgba(255, 255, 255, 0.1) inset;

			.create-input {
				width: 100%;
				padding-right: 15px;
				flex-shrink: 1;
				color: #add8e6;
			}

			.craete-button {
				display: flex;
				align-items: center;
				justify-content: center;
				flex-shrink: 0;
				width: 80px;
				height: 50px;
				border-radius: 50px;
				font-size: 16px;
				color: #88d4ec;
				box-shadow: -2px 0px 2px 1px rgba(0, 0, 0, 0.1);
			}
		}

		.create-content-box:after {
			content: '';
			position: absolute;
			right: 0;
			left: 0;
			bottom: -10px;
			margin: 0 auto;
			width: 20px;
			height: 20px;
			background: #deeff5;
			transform: rotate(45deg);
		}
	}

	.create-content:after {
		z-index: -1;
		content: '';
		position: absolute;
		right: 0;
		left: 0;
		bottom: -10px;
		margin: 0 auto;
		width: 20px;
		height: 20px;
		background: #deeff5;
		transform: rotate(45deg);
		box-shadow: -1px 1px 5px 2px rgba(0, 0, 0, 0.1);
	}

	.default {
		padding-top: 100px;

		.default-image {
			display: flex;
			justify-content: center;

			image {
				width: 100%;
			}
		}

		.default-text {
			text-align: center;
			font-size: 14px;
			color: #CCCCCC;
		}
	}

	.icon-jia {
		transition: transform 0.3s;
	}

	.create-todo-active {
		transform: rotate(135deg);
	}

	.create-show {
		opacity: 1;
		transform: scale(1) translateY(0);
	}
</style>