<template>
	<view>
		<uni-nav-bar left-icon="back" :fixed="true" :status-bar="true" title="轨迹" @clickLeft="goBack"></uni-nav-bar>
		<view class="page">
			<view class="timearea">
				<view class="time" @tap="setTime(1)">
					{{start_time}}
				</view>
				<text>至</text>
				<view class="time" @tap="setTime(0)">
					{{end_time}}
				</view>
				<view class="search-btn" @tap="getList()">搜索</view>
			</view>
			<!-- "machinefans": {
			"alias": "好吃吃就",
			"deviceid": "626160003568108"
		} -->
			<view>
				<view class="event-list " v-for="(item,index) in list" :key="index" @tap="goDetails">
					<image class="car-icon" src="../../static/image/car@2x.png"></image>
					<!-- <text class="car-name" v-if="item.machinefans.alias!=null">{{item.machinefans.alias}}</text> -->
					<text class="time">{{item.create_time}}</text>
					<view class="address">
						{{item.address}}
					</view>
				</view>
			</view>
			<view class="mask" v-if="loading">
				<text>加载中...</text>
			</view>
			<view class="mask" v-if="list.length===0 && !loading">
				<text>无数据</text>
			</view>
		</view>
		<date-time ref="dateTime" startYear="2019" :isAll="true" :current="false" @confirm="onConfirm"></date-time>
	</view>
</template>
<script>
	var _this;
	import dateTime from '@/components/yu-datetime-picker/yu-datetime-picker.vue'
	import uniSection from '@/components/uni-section/uni-section.vue'
	import uniIcons from '@/components/uni-icons/uni-icons.vue'
	import common from 'common.js'
	export default {
		components: {
			uniSection,
			uniIcons,
			dateTime
		},
		data() {
			return {
				start_time: '2020-02-28 14:09:13',
				end_time: '2020-03-09 04:44:49',
				times: '2020-03-10 08:30:01',
				type: 1,
				list: [],
				deviceid: '',
				loading: false,
				deviceType:1
			}
		},
		onLoad(e) {
			var _this = this;
			_this.deviceid = e.deviceid;
			this.deviceType = e.type
			console.log(e)
			common.request("machine/get_time_range", {
				token: uni.getStorageSync('token')
			}, function(res) {
				console.log(res)
				_this.start_time = res.data.info.end_time
				_this.end_time = res.data.info.end_time
			});
		},
		mounted(e) {

			_this = this;
			_this.getList()

		},
		methods: {
			setTime(type) {
				_this.type = type;
				if (type) {
					_this.times = _this.start_time
				} else {
					_this.times = _this.end_time
				}

				_this.$refs.dateTime.show();

			},
			getList() {
				_this.loading = true
				let data = {
					start_time: _this.start_time,
					end_time: _this.end_time,
					deviceid: _this.deviceid
				}
				common.request("machine/trajectory", data, function(res) {
					console.log(res)
					_this.list = res.data.info.list.data
					_this.loading = false
				});
			},
			onConfirm(e) {
				if (_this.type) {
					_this.start_time = e.selectRes
				} else {
					_this.end_time = e.selectRes
				}
			},
			goBack() {
				uni.navigateBack({
					data: 1
				})
			},
			goDetails() {
				var token = uni.getStorageSync('token');
				console.log(token)
				// start_time:_this.start_time,
				// end_time:_this.end_time,
				uni.navigateTo({
					url: '/pages/trajectory/details?token=' + token + "&start_time=" + _this.start_time + "&end_time=" + _this.end_time +
						'&deviceid=' + _this.deviceid + '&type=' + this.deviceType
				})
			}
		}
	}
</script>
<style lang="less" scoped>
	.uni-transition {
		bottom: calc(var(--window-bottom) + 0);
	}

	.page {
		padding-top: 100rpx;
	}

	@import url("./css/main.less");
</style>
