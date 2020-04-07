<template>
	<view class="content">
		<view class="header">
			<view class="option">
				<view class="item action">等额本息</view>
				<view class="item">等额本金</view>
			</view>
			<view class="monthly">
				<view class="text">每月月供参考（元）</view>
				<view class="price">3,205.23</view>
			</view>
			<view class="otherPrice">
				<view class="interest">
					<view class="text">支付利息（元）</view>
					<view class="price">323,205.23</view>
				</view>
				<view class="amount">
					<view class="text">还款总额（元）</view>
					<view class="price">1,233,205.23</view>
				</view>
			</view>
		</view>
		<view class="parameter">
			<view class="detailText">等额本息还款详情 ></view>
			<view class="row">
				<view class="item">
					<view class="text">贷款类型</view>
					<view class="value jiantou">
						<picker @change="bindTypeChange" :value="typeIndex" :range="typeArray">
							<view class="uni-input">{{ typeArray[typeIndex] }}</view>
						</picker>
					</view>
				</view>
				<view class="item">
					<view class="text">贷款金额（万）</view>
					<view class="value">50</view>
				</view>
				<view class="item">
					<view class="text">期限（年）</view>
					<view class="value jiantou">20</view>
				</view>
				<view class="item">
					<view class="text">利率（%）</view>
					<view class="value jiantou">4.9</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			typeArray: ['商业贷款', '公积金贷款', '组合贷款'],
			typeIndex: 0
		};
	},
	onLoad() {},
	methods: {
		bindTypeChange: function(e) {
			console.log('picker发送选择改变，携带值为', e.target.value);
			this.typeIndex = e.target.value;
		}
	}
};
</script>

<style lang="less">
@import 'index.less';
</style>
