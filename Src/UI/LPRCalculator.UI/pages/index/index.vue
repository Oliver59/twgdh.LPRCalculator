<template>
	<view class="content">
		<view class="header">
			<view class="option">
				<view class="item" :class="{ action: repaymentType == 1 }" v-on:click="bindRepaymentTypeChange(1)">等额本息</view>
				<view class="item" :class="{ action: repaymentType == 2 }" v-on:click="bindRepaymentTypeChange(2)">等额本金</view>
			</view>
			<view class="mainPrice">
				<view class="monthly">
					<view class="text">每月月供参考（元）</view>
					<view class="price">{{ this.toMoney(this.calcuteResult ? this.calcuteResult.yuegong : 0) }}</view>
				</view>
				<view class="decrease" v-if="repaymentType == 2">
					<view class="text">每月递减（元）</view>
					<view class="price">{{ this.toMoney(this.calcuteResult ? this.calcuteResult.yuegongdijian : 0) }}</view>
				</view>
			</view>
			<view class="otherPrice">
				<view class="interest">
					<view class="text">支付利息（元）</view>
					<view class="price">{{ this.toMoney(this.calcuteResult ? this.calcuteResult.totalLixi : 0) }}</view>
				</view>
				<view class="total">
					<view class="text">还款总额（元）</view>
					<view class="price">{{ this.toMoney(this.calcuteResult ? this.calcuteResult.totalPrice : 0) }}</view>
				</view>
			</view>
		</view>
		<view class="parameter">
			<view class="detailText none" v-if="this.resultIsNull()">{{ this.getLinkText() }}</view>
			<view class="detailText" v-else v-on:click="goDetail()">{{ this.getLinkText() }} ></view>
			<view class="row">
				<view class="item">
					<view class="text">贷款类型</view>
					<view class="value">
						<picker @change="bindLoansTypeChange" :value="loansTypeIndex" :range="loansTypeArray">
							<view class="uni-input">{{ loansTypeArray[loansTypeIndex] }}</view>
						</picker>
					</view>
					<text class="iconfont icon-tubiao- jiantou"></text>
				</view>
				<view class="item" v-if="loansTypeIndex != 1">
					<view class="text" v-if="loansTypeIndex == 0">贷款金额(万)</view>
					<view class="text" v-if="loansTypeIndex == 2">商款金额(万)</view>
					<input class="value" placeholder="请输入金额" @input="onKeyInput_sd" type="number" :value="syLoanAmt" />
				</view>
				<view class="item" v-if="loansTypeIndex != 0">
					<view class="text">公积金贷款金额(万)</view>
					<input class="value" placeholder="请输入金额" @input="onKeyInput_gjj" type="number" :value="gjjLoanAmt" />
				</view>
				<view class="item">
					<view class="text">期限(年)</view>
					<view class="value">
						<picker @change="bindLoanPeriodChange" :value="loanPeriodIndex" :range="loanPeriodArray" :range-key="'text'">
							<view class="uni-input">{{ loanPeriodArray[loanPeriodIndex].text }}</view>
						</picker>
					</view>
					<text class="iconfont icon-tubiao- jiantou"></text>
				</view>

				<view class="item" v-if="loansTypeIndex != 1">
					<view class="text" v-if="loansTypeIndex == 0">利率(%)</view>
					<view class="text" v-if="loansTypeIndex == 2">商贷利率(%)</view>
					<view class="value">
						<picker @change="bindInterestRateChange" :value="interestRateIndex" :range="interestRateArray" :range-key="'text'">
							<view class="uni-input">{{ this.toMoney(interestRateArray[interestRateIndex].value * this.loanPeriodArray[this.loanPeriodIndex].interest_sd) }}</view>
						</picker>
					</view>
					<text class="iconfont icon-tubiao- jiantou"></text>
				</view>
				<view class="item" v-if="loansTypeIndex != 0">
					<view class="text" v-if="loansTypeIndex == 1">利率(%)</view>
					<view class="text" v-if="loansTypeIndex == 2">公积金贷利率(%)</view>
					<view class="value">
						<picker @change="bindInterestRateChange_Gjj" :value="interestRateIndex_Gjj" :range="interestRateArray_Gjj" :range-key="'text'">
							<view class="uni-input">
								{{ this.toMoney(interestRateArray_Gjj[interestRateIndex_Gjj].value * this.loanPeriodArray[this.loanPeriodIndex].interest_gjj) }}
							</view>
						</picker>
					</view>
					<text class="iconfont icon-tubiao- jiantou"></text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
import calcute from '@/pages/index/calculate.js';
export default {
	data() {
		return {
			repaymentType: 1, //1等额本息 2等额本金
			syLoanAmt: '', //商业贷款金额
			gjjLoanAmt: '', //公积金贷款金额
			loansTypeArray: ['商业贷款', '公积金贷款', '组合贷款'],
			loansTypeIndex: 0,
			loanPeriodArray: [
				{ text: 5, interest_sd: 4.75, interest_gjj: 2.75 },
				{ text: 10, interest_sd: 4.9, interest_gjj: 3.25 },
				{ text: 15, interest_sd: 4.9, interest_gjj: 3.25 },
				{ text: 20, interest_sd: 4.9, interest_gjj: 3.25 },
				{ text: 25, interest_sd: 4.9, interest_gjj: 3.25 },
				{ text: 30, interest_sd: 4.9, interest_gjj: 3.25 }
			],
			loanPeriodIndex: 0,
			interestRateArray: [
				{ text: '基准利率', value: 1 },
				{ text: '7折利率', value: 0.7 },
				{ text: '8折利率', value: 0.8 },
				{ text: '8.3折利率', value: 0.83 },
				{ text: '8.5折利率', value: 0.85 },
				{ text: '8.8折利率', value: 0.88 },
				{ text: '9折利率', value: 0.9 },
				{ text: '9.5折利率', value: 0.95 },
				{ text: '1.05倍利率', value: 1.05 },
				{ text: '1.1倍利率', value: 1.1 },
				{ text: '1.2倍利率', value: 1.2 },
				{ text: '1.3倍利率', value: 1.3 },
				{ text: '1.4倍利率', value: 1.4 }
			],
			interestRateIndex: 0,
			interestRateArray_Gjj: [{ text: '基准利率', value: 1 }, { text: '1.1倍利率', value: 1.1 }],
			interestRateIndex_Gjj: 0,
			calcuteResult: null
		};
	},
	onLoad() {},
	methods: {
		bindRepaymentTypeChange: function(value) {
			if (this.repaymentType == value) return;
			this.repaymentType = value;
			this.calculateLoan();
		},
		// 改变贷款类型事件
		bindLoansTypeChange: function(e) {
			if (e.target.value == this.loansTypeIndex) return;
			this.loansTypeIndex = parseInt(e.target.value);
			this.monthlyAmt = 0; //月供
			this.decreaseAmt = 0; //每月递减额
			this.interestAmt = 0; //利息
			this.totalAmt = 0; //还款总额
			this.syLoanAmt = ''; //商业贷款金额
			this.gjjLoanAmt = ''; //商业贷款金额
			this.calcuteResult = null;
		},
		// 改变贷款期限事件
		bindLoanPeriodChange: function(e) {
			this.loanPeriodIndex = e.target.value;
			this.calculateLoan();
		},
		// 改变贷款利率事件
		bindInterestRateChange: function(e) {
			this.interestRateIndex = e.target.value;
			this.calculateLoan();
		},
		// 改变贷款利率（公积金）事件
		bindInterestRateChange_Gjj: function(e) {
			this.interestRateIndex_Gjj = e.target.value;
			this.calculateLoan();
		},
		//输入商业贷款金额事件
		onKeyInput_sd: function(event) {
			var num = event.target.value;
			this.syLoanAmt = parseFloat(num);
			// num = num.toFixed(2);
			this.calculateLoan();
		},
		//输入公积金贷款金额事件
		onKeyInput_gjj: function(event) {
			var num = event.target.value;
			// num = num.toFixed(2);
			this.gjjLoanAmt = parseFloat(num);
			this.calculateLoan();
		},
		calculateLoan: function() {
			let type = this.repaymentType;
			let loanPeriod = this.loanPeriodArray[this.loanPeriodIndex];
			let year = loanPeriod.text;

			let sdnum = this.syLoanAmt;
			let gjjnum = this.gjjLoanAmt;

			let sdlilv = this.toMoney(this.interestRateArray[this.interestRateIndex].value * loanPeriod.interest_sd) / 100;
			let gjjlilv = this.toMoney(this.interestRateArray_Gjj[this.interestRateIndex_Gjj].value * loanPeriod.interest_gjj) / 100;

			if (year == 0) return;
			console.log(`type:${type}, sdnum:${sdnum}, gjjnum:${gjjnum},  year:${year}, sdlilv:${sdlilv}, gjjlilv:${gjjlilv}`);
			console.log('this.loansTypeIndex', this.loansTypeIndex);
			console.log(typeof this.loansTypeIndex);
			this.calcuteResult = null;
			switch (this.loansTypeIndex) {
				case 0:
					// 商业贷款
					console.log('商业贷款start····');
					if (sdnum == 0 || sdlilv == 0) return;
					this.calcuteResult = calcute.singleDk(type, sdnum, year, sdlilv);
					break;
				case 1:
					// 公积金贷款
					console.log('公积金贷款start····');
					if (gjjnum == 0 || gjjlilv == 0) {
						console.log('跳出公积金贷款');
						return;
					}
					console.log('执行start····');
					this.calcuteResult = calcute.singleDk(type, gjjnum, year, gjjlilv);
					console.log('执行完毕');
					break;
				case 2:
					// 组合贷款
					console.log('组合贷款start····');
					if (!this.syLoanAmt || !this.gjjLoanAmt) return;
					this.calcuteResult = calcute.zuhe(type, sdnum, gjjnum, year, year, sdlilv, gjjlilv);
					break;
			}
			console.log('this.calcuteResult', this.calcuteResult);
			console.log('this.calcuteResult==null', this.calcuteResult == null);
		},
		goDetail: function() {
			let type = this.repaymentType;
			let loanPeriod = this.loanPeriodArray[this.loanPeriodIndex];
			let year = loanPeriod.text;

			let sdnum = this.syLoanAmt;
			let gjjnum = this.gjjLoanAmt;

			let sdlilv = this.toMoney(this.interestRateArray[this.interestRateIndex].value * loanPeriod.interest_sd) / 100;
			let gjjlilv = this.toMoney(this.interestRateArray_Gjj[this.interestRateIndex_Gjj].value * loanPeriod.interest_gjj) / 100;

			uni.navigateTo({
				url: `../detail/detail?loansTypeIndex=${this.loansTypeIndex}&repaymentType=${type}&year=${year}&sdnum=${sdnum}&gjjnum=${gjjnum}&sdlilv=${sdlilv}&gjjlilv=${gjjlilv}`
			});
		},
		resultIsNull: function() {
			return this.calcuteResult == null;
		},
		getLinkText: function() {
			// 1等额本息 2等额本金
			let type = '等额本息';
			if (this.repaymentType == 2) {
				type = '等额本金';
			}
			return type + '还款详情';
		},
		// 将数字转化为金额类型
		toMoney2: function(num) {
			num = num.toFixed(2);
			num = parseFloat(num);
			num = num.toLocaleString();
			return num; //返回的是字符串23,245.12保留2位小数
		},
		toMoney: function(number) {
			var decimals = 2;
			var dec_point = '.';
			var thousands_sep = ',';
			/*
			 * 参数说明：
			 * number：要格式化的数字
			 * decimals：保留几位小数
			 * dec_point：小数点符号
			 * thousands_sep：千分位符号
			 * */
			number = (number + '').replace(/[^0-9+-Ee.]/g, '');
			var n = !isFinite(+number) ? 0 : +number,
				prec = !isFinite(+decimals) ? 0 : Math.abs(decimals),
				sep = typeof thousands_sep === 'undefined' ? ',' : thousands_sep,
				dec = typeof dec_point === 'undefined' ? '.' : dec_point,
				s = '',
				toFixedFix = function(n, prec) {
					var k = Math.pow(10, prec);
					return '' + Math.floor(n * k) / k;
				};
			s = (prec ? toFixedFix(n, prec) : '' + Math.floor(n)).split('.');
			var re = /(-?\d+)(\d{3})/;
			console.log(s);
			while (re.test(s[0])) {
				s[0] = s[0].replace(re, '$1' + sep + '$2');
			}

			if ((s[1] || '').length < prec) {
				s[1] = s[1] || '';
				s[1] += new Array(prec - s[1].length + 1).join('0');
			}
			return s.join(dec);
		}
	}
};
</script>

<style lang="less">
@import '~@/static/css/iconfont.css';
@import 'index.less';
</style>
