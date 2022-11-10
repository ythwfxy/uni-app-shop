<template>
	<view class="login-container">
		<uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
		<button type="primary" class="btn-login" @click="getUserProfile">一键登录</button>
		<text class="tips-text">登陆后尽享更多权益</text>
	</view>
</template>

<script>
	import {mapMutations,mapState} from 'vuex'
	export default {
		name:"my-login",
		data() {
			return {
				
			};
		},
		computed:{
			...mapState('m_user',['redirectInfo'])
		},
		methods:{
			...mapMutations('m_user',['updateUserInfo','updateToken','updateRedirectInfo']),
			getUserProfile(){
				uni.getUserProfile({
					desc:'userinfo',
					success:res=>{
						this.updateUserInfo(res.userInfo)
						this.getToken(res)
					},
					fail:err=>{
						uni.$showMsg('您取消了授权')
					}
				})
			},
			async getToken(info){
				const [err,res]= await uni.login().catch(err=>err)
				if(err||res.errMsg!="login:ok") return uni.$showMsg('登录失败')
				const query={
					code:res.code,
					encryptedData:info.encryptedData,
					iv:info.iv,
					rawData:info.rawData,
					signature:info.signature
				}
				const {data:loginResult} =await uni.$http.post('/api/public/v1/users/wxlogin',query)
				// if(loginResult.meta.status!==200) return uni.$showMsg('登录失败')
				// loginResult.message.token
				this.updateToken('mytoken')
				this.navigateBack()
			},
			navigateBack(){
				if(this.redirectInfo && this.redirectInfo.openType==='switchTab'){
					uni.switchTab({
						url:this.redirectInfo.from,
						complete:()=>{
							this.updateRedirectInfo(null)
						}
					})
				}
			}
		}
	}
</script>

<style lang="scss">
.login-container{
	display: flex;
	height: 750rpx;
	background-color: #F8F8F8;
	
	flex-direction: column;
	justify-content: center;
	align-items: center;
	position: relative;
	overflow: hidden;
	&::after{
		content: ' ';
		display: block;
		width: 100%;
		height: 40px;
		position: absolute;
		bottom: 0;
		left: 0;
		border-radius: 100%;
		transform: translateY(50%);
		background-color: white;
	}
	.btn-login{
		width: 90%;
		border-radius: 100px;
		margin: 15px 0;
		background-color: #C00000;
	}
	.tips-text{
		font-size: 12px;
		color:gray
	}
}
</style>