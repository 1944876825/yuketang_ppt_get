<template>
	<view class="content flex-col">
		<view class="header" style="height: 140rpx;">
			<view style="position: fixed;top: 0;width: 100%;z-index: 1;background-color: #fff;">
				<view class="">
					<scroll-view style="white-space: nowrap;" :scroll-x="true" :show-scrollbar="false" scroll-with-animation>
						<view class="options">
							<view 
								v-for="(title, index) in slideTitleList"
								class="obox"
								:class="slideIndex == index ? 'actobox' : ''"
								@tap="chooseSlideIndex(index)">
								<text style="">{{ title }}</text>
							</view>
						</view>
					</scroll-view>
				</view>
				<view class="options">
					<view class="obox" :class="dqxz == 'ALL' ? 'actobox' : ''" @tap="choptions('ALL')">
						全部
					</view>
					<view class="obox" :class="dqxz == 'PPT' ? 'actobox' : ''" @tap="choptions('PPT')">
						PPT
					</view>
					<view class="obox" :class="dqxz == 'Question' ? 'actobox' : ''" @tap="choptions('Question')">
						题目
					</view>
					<view class="obox" :class="isSave == true ? 'actobox' : ''" @tap="unsave()">
						删除
					</view>
				</view>
			</view>
		</view>
		
		<view class="center" style="margin-top: 8px;">
			<scroll-view scroll-y="true" v-if="isShowPPT">
				<view v-for="(slide, index) in slideList[slideIndex]" class="pptBox flex-col items-center" v-if="dqxz=='ALL' || (slide['problem'] && dqxz=='Question') || (!slide['problem'] && dqxz=='PPT')">
					<image :id="'fg'+slide.id" :src="slide['cover']" mode="aspectFit" @click="imgShow(index)"></image>
					<block v-if="slide['problem'] && slide['problem']['answers'] && slide['problem']['answers']['length']>0">
						<text class="answer" style="font-size: 16px;font-weight: 800;">答案：{{slide['problem']['answers'].join(' ')}}</text>
					</block>
					<text style="font-size: 13px;font-weight: 600;margin-top: 10rpx;">第{{slide['index']}}张</text>
				</view>
			</scroll-view>
			<view class="tips" v-else>
				{{ jztips }}
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				source: '',
				lessonId: '',
				presentation_id_list: [],
				imgList: [],
				authHeader: '',
				websocket: {},
				userid: '',
				wssAuth: '',
				dqxz: 'ALL',
				courseName: '',
				pptList: {},
				baseInfo: {},// 当前课程信息
				isShowPPT: false,// 是否显示PPT
				
				slideIndex: 0,
				slideTitleList: [],
				slideList: [],
				// currentSlideList: [],
				jztips: '拼命加载中...'
			};
		},
		onLoad(opt) {
			this.slideTitleList = []
			this.slideList = []
			this.load()
		},
		onPullDownRefresh() {
			this.slideTitleList = []
			this.slideList = []
			this.load()
			setTimeout(function() {
				uni.stopPullDownRefresh();
			}, 500);
		},
		methods: {
			unsave() {
				var pptList = uni.getStorageSync('pptList')
				delete(pptList[this.lessonId])
				uni.setStorage({
					key: 'pptList',
					data: pptList,
					success:ress=>{
						this.isSave = false
						this.pptList = pptList
						uni.showToast({
							icon:'none',
							title:'取消成功'
						})
						uni.navigateBack()
					},
					fail:errr=>{
						this.isSave = true
						uni.showToast({
							icon:'none',
							title:'取消失败'
						})
					}
				})
			},
			load() {
				const option = uni.getStorageSync('current_class')
				if (!option) {
					uni.showToast({
						icon:'none',
						title:'缓存获取失败'
					})
					return
				}
				this.courseName = option['courseName']
				this.source = option['role']
				this.lessonId = option['lessonId']
				if (option['title'] && option['title']!='') {
					this.baseInfo['title'] = option['title']
					uni.setNavigationBarTitle({
						title: this.baseInfo['title']
					})
				}
				this.getHistory()
			},
			getHistory() {
				const res = uni.getStorageSync('pptList')
				if (res) {
					this.pptList = res
					this.slideTitleList = this.pptList[this.lessonId]['slideList']['title']
					this.slideList = this.pptList[this.lessonId]['slideList']['list']
					this.chooseSlideIndex(this.slideIndex)
					if (this.slideList.length > 0) {
						this.isShowPPT = true
						var imgList = []
						this.slideList.forEach(list=>{
							var all_cover = []
							list.forEach(e=>{
								all_cover.push(e['cover'])
							})
							imgList.push(all_cover)
						})
						this.imgList = imgList
					}
				}
			},
			// 切换ppt index
			chooseSlideIndex(index) {
				// this.currentSlideList = this.slideList[index].slice(0, 10)
				this.slideIndex = index
			},
			choptions(option) {
				this.dqxz = option
				uni.pageScrollTo({
					scrollTop: 0,
					duration: 300
				});
			},
			imgShow(cindex) {
				uni.previewImage({
					current: cindex,
					urls: this.imgList[this.slideIndex],
					loop: true
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.header {
		.options {
			display: flex;
			flex-direction: row;
			flex-wrap: nowrap;
			.actobox {
				color: #fff !important;
				background-color: $yuKeTang;
			}
			.obox {
				padding: 10rpx 20rpx;
				margin: 10rpx;
				border: 1px solid $yuKeTang;
				border-radius: 8rpx;
				color: $yuKeTang;
				font-size: 13px;
			}
		}
	}
	.center {
		.pptList {
			padding: 15rpx 20rpx;
		}
	}
	.pptBox {
		margin: 10px;
		border-radius: 20rpx;
		box-shadow: 1px 1px 1px 1px #DEEAF7;
		background-color: #fff;
		padding: 8px;
		.answer {
			color: $yuKeTang;
		}
	}
	.tips {
		text-align: center;
		color: $yuKeTang;
		font-size: 14px;
		font-weight: 600;
		margin: 30rpx 0;
	}
</style>
