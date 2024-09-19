<template>
	<view class="tuixiu-page">
		<view class="txp-top">
			<view class="title">法定退休年龄计算器</view>
			<view class="subtitle">-2024年9月13日新规</view>
			<image class="img" src="/static/home/延迟退休_cover.png" mode="widthFix"></image>
		</view>
		<view class="tx-section">
			<uni-forms ref="form" :modelValue="formData" label-position="left">
				<picker mode="date" fields="month" v-model="formData.birthday"
					@change="handleChangeDate">
					<view class="form-item">
						<view class="form-item-label">
							<text class="red">*</text>
							出生日期
						</view>
						<view class="form-item-content">
							<text v-if="formData.birthday">{{formData.birthday}}</text>
							<text v-else>请选择 &gt;</text>
						</view>
					</view>
				</picker>
				<picker mode="selector" :range="genderList" v-model="formData.birthday" @change="handleChangeGender">
					<view class="form-item">
						<view class="form-item-label">
							<text class="red">*</text>
							性别及人员类型
						</view>
						<view class="form-item-content">
							<text v-if="formData.gender">{{genderList[formData.gender]}}</text>
							<text v-else>请选择 &gt;</text>
						</view>
					</view>
				</picker>
				<view style="text-align: center;">
					<button type="primary" :loading="!showAnswer" class="tx-submit" @click="handleSubmit">计算</button>
				</view>
			</uni-forms>
		</view>
		<view class="tx-section txp-desc">
			<text>说明：按照</text>
			<text class="uni-link">《关于实施渐进式延迟法定退休年龄的决定》附表对照关系</text>
			<text>，您通过法定退休年龄计算器，选择出生年月、性别及人员类型，即可计算出对应的改革后法定退休年龄、改革后退休时间、延迟月数。</text>
		</view>
		<view v-if="showAnswer" class="tx-section txp-result">
			<view class="txp-result-line">计算结果</view>
			<view class="txp-result-item">
				<view class="left">职工类型：</view>
				<view class="right">{{calcResult.gender}}</view>
			</view>
			<view class="txp-result-item">
				<view class="left">出生时间：</view>
				<view class="right">{{calcResult.birthday}}</view>
			</view>
			<view class="txp-result-item">
				<view class="left">您的改革后法定退休年龄为：</view>
				<view class="right">{{calcResult.reformedRetirementAge}}</view>
			</view>
			<view class="txp-result-item">
				<view class="left">您的改革后退休时间为：</view>
				<view class="right">{{calcResult.reformedRetirementTime}}</view>
			</view>
			<view class="txp-result-item">
				<view class="left">您的延迟月数为：</view>
				<view class="right">{{calcResult.delayMonths}}</view>
			</view>
		</view>
		<!-- 提示信息弹窗 -->
		<uni-popup ref="message" type="message">
			<uni-popup-message type="error" :message="errorMsg" :duration="2000"></uni-popup-message>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				showAnswer: true,
				errorMsg: '',
				currentDate: new Date(),
				genderList: ["男职工", "原法定退休年龄55周岁女职工", "原法定退休年龄50周岁女职工"],
				formData: {
					birthday: '',
					gender: '',
				},
				calcResult: {
					birthday: '',
					gender: '',
					reformedRetirementAge: '',
					reformedRetirementTime: '',
					delayMonths: ''
				}
			};
		},
		onShareAppMessage() {
			return {
				title: '2024年9月最新版退休计算器',
				path: '/pages/tuixiu/tuixiu'
			}
		},
		methods: {
			handleChangeDate(e) {
				this.formData.birthday = e.detail.value
			},
			handleChangeGender(e) {
				this.formData.gender = e.detail.value
			},
			handleSubmit() {
				if (this.formData.birthday == null || this.formData.birthday == '') {
					this.errorMsg = "'出生日期'不能为空";
					this.$refs.message.open();
					return;
				}
				if (this.formData.gender == null || this.formData.gender == '') {
					this.errorMsg = "'性别及人员类型'不能为空";
					this.$refs.message.open();
					return;
				}
				this.showAnswer = false;
				let birthday = this.formData.birthday;
				let gender = this.formData.gender;
				this.calculateRetirementAge(birthday, gender);
				setTimeout(() => {
					this.showAnswer = true;
				}, 800);
			},

			/**
			 * 计算延迟退休年龄
			 * 男职工：
			 *   - 自1965年1月起，至1976年12月止，每4个月延迟1个月退休
			 * 原法定退休年龄55周岁女职工：
			 *   - 自1970年1月起，至1981年12月止，每4个月延迟1个月退休
			 * 原法定退休年龄50周岁女职工：
			 *   - 自1975年1月起，至1984年12月止，每2个月延迟1个月退休
			 * @param {Object} birthDate 出生时间，如"1975-06"
			 * @param {Object} personInfo 性别及人员类型，如“男职工”
			 */
			calculateRetirementAge(birthDate, personInfo) {
				// “男职工”延迟退休的开始时间、结束时间、原退休年龄、延迟时间
				let maleDelayRetireStart = new Date("1965-01-01");
				let maleDelayRetireEnd = new Date("1976-12-01");
				let maleRetireBaseYear = 60;
				let maleRetireStep = 4;

				// “原法定退休年龄55周岁女职工”延迟退休的开始时间、结束时间、原退休年龄、延迟时间
				let female55DelayRetireStart = new Date("1970-01-01");
				let female55DelayRetireEnd = new Date("1981-12-01");
				let female55RetireBaseYear = 55;
				let female55RetireStep = 4;

				// “原法定退休年龄50周岁女职工”延迟退休的开始时间、结束时间、原退休年龄、延迟时间
				let female50DelayRetireStart = new Date("1975-01-01");
				let female50DelayRetireEnd = new Date("1984-12-01");
				let female50RetireBaseYear = 50;
				let female50RetireStep = 2;

				// 计算“男职工”的退休时间
				if (personInfo == 0 || personInfo == "男职工") {
					this.calc(birthDate, maleDelayRetireStart, maleDelayRetireEnd, maleRetireBaseYear, maleRetireStep);
				}
				// 计算“原法定退休年龄55周岁女职工”的退休时间
				else if (personInfo == 1 || personInfo == "原法定退休年龄55周岁女职工") {
					this.calc(birthDate, female55DelayRetireStart, female55DelayRetireEnd, female55RetireBaseYear,
						female55RetireStep);
				}
				// 计算“原法定退休年龄50周岁女职工”的退休时间
				else if (personInfo == 2 || personInfo == "原法定退休年龄50周岁女职工") {
					this.calc(birthDate, female50DelayRetireStart, female50DelayRetireEnd, female50RetireBaseYear,
						female50RetireStep);
				} else {
					this.errorMsg = "未知的‘性别及人员类型’";
					this.$refs.message.open();
					return;
				}
			},
			calc(birthDate, delayRetireStart, delayRetireEnd, retireBaseYear, retireStep) {
				// 定义返回结果
				let birthday = new Date(birthDate);
				var delayMonths = 0; // 延迟月数

				// 早于延迟退休时间，返回原退休时间
				if (birthday < delayRetireStart) {
					delayMonths = 0;
				}
				// 晚于渐进式延迟退休时间，返回最大退休时间
				else if (birthday > delayRetireEnd) {
					delayMonths = 36;
				}
				// 按照渐进式延迟时间计算退休时间
				else {
					while (delayRetireStart <= birthday) {
						delayMonths++;
						delayRetireStart.setMonth(delayRetireStart.getMonth() + retireStep);
					}
				}

				let retireDay = new Date(birthDate);
				retireDay.setFullYear(retireDay.getFullYear() + retireBaseYear);
				retireDay.setMonth(retireDay.getMonth() + delayMonths);

				this.calcResult = {
					birthday: `${birthday.getFullYear()}年${birthday.getMonth() +1}月`,
					gender: this.genderList[this.formData.gender],
					reformedRetirementAge: this.transReformedRetirementAge(birthday, retireDay),
					reformedRetirementTime: `${retireDay.getFullYear()}年${retireDay.getMonth() + 1} 月`,
					delayMonths: delayMonths
				}
			},
			transReformedRetirementAge(birthday, retireday) {
				var years = retireday.getFullYear() - birthday.getFullYear();
				var months = retireday.getMonth() - birthday.getMonth();
				if (months < 0) {
					years--;
					months += 12;
				}
				var result = `${years}岁`;
				if (months > 0) {
					result += `${months}个月`;
				}
				return result;
			},
		}
	}
</script>

<style lang="scss">
	.tuixiu-page {
		font-size: 14px;
		line-height: 24px;

		.tx-section {
			background-color: #FFFFFF;
			padding: 30rpx 30rpx;
			border-radius: 10rpx;
			box-shadow: 0px 0px 2px #dddddd;
			margin-bottom: 20rpx;
		}

		.txp-top {
			background-color: #FFFFFF;
			padding: 50rpx 30rpx;
			margin-bottom: 20rpx;
			position: relative;
			overflow: hidden;

			.title {
				color: #5079e6;
				font-size: 30px;
				margin-bottom: 30rpx;
				font-weight: bold;
			}
			
			.subtitle {
				color: #7e8186;
			}
			
			.img {
				width: 300rpx;
				position: absolute;
				top: 0;
				right: -40rpx;
			}
		}

		.tx-submit {
			width: 400rpx;
			height: 80rpx;
			line-height: 80rpx;
			background-color: #5079e6;
		}

		.form-item {
			display: flex;
			justify-content: space-between;
			height: 60rpx;
			margin-bottom: 30rpx;

			.form-item-label {
				text-align: left;
				font-size: 14px;
				color: #606266;
				height: 36px;

				.red {
					color: red;
				}
			}

			.form-item-content {}
		}

		.txp-desc {
			font-size: 12px;
			color: #606266;
			line-height: 40rpx;
			padding: 20rpx;
			border-radius: 10rpx;
		}

		.uni-link {
			color: #55aaff;
		}

		@keyframes scaleAnimation {
			0% {
				height: 0px;
			}

			50% {
				height: 200rpx;
			}

			70% {
				height: 300rpx;
			}

			100% {
				height: 100%;
			}
		}

		.txp-result {
			overflow: hidden;
			border-radius: 20rpx;
			text-align: left;
			padding: 50rpx 30rpx 30rpx;
			line-height: 70rpx;
			color: #606266;
			position: relative;
			animation: scaleAnimation 2s 1;

			.txp-result-line {
				height: 60rpx;
				line-height: 80rpx;
				text-align: center;
				background-color: #5079e6;
				border-radius: 20rpx;
				box-shadow: 3px 2px #606266;
				margin-bottom: 30rpx;
				color: #FFFFFF;
				position: absolute;
				top: -20rpx;
				left: 250rpx;
				width: 250rpx;
			}

			.txp-result-item {
				display: flex;
				justify-content: space-between;

				.right {
					color: black;
					font-size: 16px;
				}
			}

		}
	}
</style>