<template>
	<view class='content'>
		<view>
			<u--form labelWidth='60' labelPosition="left" :model="model" :rules="rules" ref="form1">
				<u-form-item label="用户名" prop="userInfo.name" borderBottom ref="item1">
					<u--input v-model="model.userInfo.name" placeholder='请输入用户名' border="none"></u--input>
				</u-form-item>
				<u-form-item label="密码" prop="userInfo.password" borderBottom ref="item1">
					<u--input :type='passwordType' placeholder='请输入密码' v-model="model.userInfo.password" border="none">
					</u--input>
					<span>
						<image @click="showPwd" :src="imgChange" mode=""></image>
					</span>
				</u-form-item>
				<u-form-item label="手机号" prop="userInfo.phone" borderBottom ref="item1">
					<u--input v-model="model.userInfo.phone" placeholder='请输入手机号码' border="none"></u--input>
				</u-form-item>
				<u-form-item label="验证码" prop="userInfo.code" borderBottom ref="item1">
					<u--input v-model="model.userInfo.code" placeholder='请输入验证码' border="none"></u--input>
					<view class="code">
						<!-- <u-button type="primary"  @click="isCode=!isCode" v-if="isCode">
							获取验证码
						</u-button>
						<u-button type="info" v-else class="countDown">
							<u-count-down
							        :time="10 * 1000"
							        format="ss"
							        autoStart
									@finish="isCode=!isCode"
							        millisecond
							    >
							    </u-count-down>s后重新发送
						</u-button>
					 -->
						<view class="wrap">
							<u-toast ref="uToast"></u-toast>
							<u-code :seconds="seconds" @end="end" @start="start" ref="uCode" @change="codeChange">
							</u-code>
							<u-button @tap="getCode">{{tips}}</u-button>
						</view>
					</view>
				</u-form-item>
				<u-form-item label="性别" prop="userInfo.sex" borderBottom
					ref="item1">
					<u-radio-group v-model="model.userInfo.sex">
						<u-radio :customStyle="{marginLeft: '16px'}" v-for="(item, index) in radiolist1" :key="index"
							:label="item.name" :name="item.name">
						</u-radio>
					</u-radio-group>
				</u-form-item>
			</u--form>

			<view class="regBut">
				<u-button class="register" @click="register" iconColor='red' type="error" size="large" text='注册'>
				</u-button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				tips: '',
				seconds: 10,//验证码时间
				passwordType: 'password',//密码输入框类型
				imgChange: '../../static/eye-close.png',//密码输入框切换图标
				verifyCode: '',//验证码
				model: {
					userInfo: {
						name: '',
						password:'',
						phone: '',
						code: '',
						sex: '男',
					},
				},
				radiolist1: [{
						name: '男',
						disabled: false
					},
					{
						name: '女',
						disabled: false
					}
				],
				rules: {
					'userInfo.name': {
						type: 'string',
						required: true,
						message: '请输入用户名',
						trigger: ['blur']
					},
					'userInfo.password': [{
						type: 'string',
						required: true,
						message: '请输入密码',
					}, {
						validator: (rule, value, callback) => {
							//密码验证规则
							let reg =
								/^(?=.*[A-Za-z]{1,})(?=.*\d)(?=.*[`~!@#$%^&*()_+<>?:"{},.\/\\;'[\]]{1,})[A-Za-z\d`~!@#$%^&*()_+<>?:"{},.\/\\;'[\]]{8,}$/
							return reg.test(value)
						},
						message: '密码格式应包含字母数字特殊字符,且密码长度大于8',
						trigger: ['blur'],
					}],
					'userInfo.phone': [{
							required: true,
							message: '请输入手机号',
							trigger: ['blur'],
						},
						{
							// 自定义验证函数，见上说明
							validator: (rule, value, callback) => {
								// 上面有说，返回true表示校验通过，返回false表示不通过
								// uni.$u.test.mobile()就是返回true或者false的
								return uni.$u.test.mobile(value);
							},
							message: '手机号码不正确',
							// 触发器可以同时用blur和change
							trigger: ['blur'],
						}
					],
					'userInfo.code': [{
							required: true,
							message: '请输入验证码',
							trigger: ['blur'],
						},
						{
							// 自定义验证函数，见上说明
							validator: (rule, value, callback) => {
								// 上面有说，返回true表示校验通过，返回false表示不通过
								// uni.$u.test.mobile()就是返回true或者false的
								if (value != this.verifyCode) return false
							},
							message: '验证码不正确，请重新输入',
							// 触发器可以同时用blur和change
							trigger: ['blur'],
						}
					],
					'userInfo.sex': {
						type: 'string',
						max: 1,
						required: true,
						message: '请选择男或女',
						trigger: ['change']
					},
				},
				radio: '',
				isCode: true
			}
		},
		onLoad() {

		},
		methods: {
			register() {
				//注册
				this.$refs.form1.validate().then(res => {
					this.testclientdb()
				}).catch(err => {
					console.log(err);
				})
			},
			testclientdb() {
				// 向数据库添加数据
				const db = uniCloud.database();
				db.collection("users").add(this.model.userInfo).then((res) => {
					// res 为数据库查询结果
					uni.showLoading({
						title: '恭喜您，注册成功'
					})
					setTimeout(() => {
						uni.hideLoading();
						uni.$u.toast('即将跳转...');
						this.$router.go()
					}, 2000);
					this.verifyCode=''
				}).catch((e) => {
					console.log(e)
				});
			},
			getDB(){
				// 查询数据库
				const db = uniCloud.database();
				db.collection("users").get().then((res) => {
					// res 为数据库查询结果
					console.log(res.result.data)
				}).catch((e) => {
					console.log(e)
				});
			},
			showPwd() {
				//控制密码输入框密码是否可见
				if (this.passwordType === 'password') {
					this.passwordType = ''
					this.imgChange = '../../static/eye.png'
				} else {
					this.passwordType = 'password'
					this.imgChange = '../../static/eye-close.png'
				}
			},
			codeChange(text) {
				this.tips = text;
			},
			getCode() {
				if (this.$refs.uCode.canGetCode) {
					// 模拟向后端请求验证码
					uni.showLoading({
						title: '正在获取验证码'
					})
					setTimeout(() => {
						uni.hideLoading();
						// 这里此提示会被this.start()方法中的提示覆盖
						uni.$u.toast('验证码已发送');
						this.getVerifyCode()
						// 通知验证码组件内部开始倒计时
						this.$refs.uCode.start();
					}, 2000);
				} else {
					uni.$u.toast('倒计时结束后再发送');
				}
			},
			end() {},
			start() {},
			getRandom(min, max) {
				//生成随机数
				return Math.floor(Math.random() * (max - min + 1)) + min
			},
			getVerifyCode() {
				//随机生成验证码
				this.verifyCode = ''
				for (let i = 0; i < 4; i++) {
					const ascii = this.getRandom(48, 112)
					if ((ascii > 57 && ascii < 65) || (ascii > 90 && ascii < 97)) {
						i--
						continue
					}
					const c = String.fromCharCode(ascii)
					this.verifyCode += c
				}
				console.log(this.verifyCode);
				setTimeout(() => {
					uni.$u.toast(`验证码：${this.verifyCode}`)
				}, 2000)
				return this.verifyCode
			}
		},
		onReady() {
			this.getDB()
		},
	}
</script>

<style>
	.content {
		width: 100%;
		height: 100%;
		background-image: url('../../static/bg.jpg');
		background-size: cover;
		background-repeat: no-repeat;
		display: flex;

	}

	.content>view {
		margin: auto;
		padding: 10px;
		border: 1px solid white;
		border-radius: 10px;
		background-color: rgba(255, 255, 255, .5);
	}


	.register {
		width: 200px;
		height: 40px;
		padding: 20px;
		border: 1px solid #F56C6C;
		border-radius: 20px;
		margin-top: 10px;
	}

	image {
		width: 24px;
		height: 24px;
		padding-right: 16px;
	}

	.code {
		border: 1px solid white;
		border-radius: 10px;
		overflow: hidden;
	}
	.wrap{
		box-sizing: border-box;
		width: 120px;
	}
</style>
