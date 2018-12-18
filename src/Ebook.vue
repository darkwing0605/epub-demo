<template>
	<div class="ebook">
		<title-bar :ifTitleShow="ifTitleShow"></title-bar>
		<div class="read-wrapper">
			<div id="read"></div>
			<div class="mask">
				<div class="left" @click="prevPage"></div>
				<div class="center" @click="toggleTitle"></div>
				<div class="right" @click="nextPage"></div>
			</div>
		</div>
		<menu-bar
			:ifTitleShow="ifTitleShow"
			:fontSizeList="fontSizeList"
			:defaultFontSize="defaultFontSize"
			@setFontSize='setFontSize'
			:themeList="themeList"
			:defaultTheme="defaultTheme"
			@setTheme="setTheme"
			:bookAvailable="bookAvailable"
			@onProgressChange="onProgressChange"
			:navigation="navigation"
			@jumpTo="jumpTo"
			:changeProgress="changeProgress"
			ref="menuBar"
		></menu-bar>
	</div>
</template>

<script>
	import titleBar from '@/components/titleBar';
	import menuBar from '@/components/menuBar';
	import Epub from 'epubjs';
	const DOWNLOAD_URL = '/MBook01.epub';
	global.epub = Epub;
	export default {
		components: {
			titleBar,
			menuBar,
		},
		data: function() {
			return {
				ifTitleShow: false,
				fontSizeList: [
					{fontSize: 12},
					{fontSize: 14},
					{fontSize: 16},
					{fontSize: 18},
					{fontSize: 20},
					{fontSize: 22},
					{fontSize: 24},
				],
				defaultFontSize: 16,
				themeList: [
					{
						name: 'default',
						style: {
							body: {
								'color': '#000',
								'background': '#fff',
							}
						}
					},
					{
						name: 'eye',
						style: {
							body: {
								'color': '#000',
								'background': '#ceeaba',
							}
						}
					},
					{
						name: 'night',
						style: {
							body: {
								'color': '#fff',
								'background': '#000',
							}
						}
					},
					{
						name: 'gold',
						style: {
							body: {
								'color': '#000',
								'background': 'rgb(241, 236, 226)',
							}
						}
					}
				],
				defaultTheme: 0,
				// 图书是否处于可用状态
				bookAvailable: false,
				navigation: {},
				progress: 0,
			}
		},
		methods: {
			// 根据链接跳转到指定位置
			jumpTo: function(href) {
				this.rendition.display(href)
				this.hideTitleAndMenu();
				this.changePage();
			},
			hideTitleAndMenu: function() {
				// 隐藏标题栏和菜单栏
				this.ifTitleShow = false;
				// 隐藏菜单栏弹出的设置栏
				this.$refs.menuBar.hideSetting()
				// 隐藏目录
				this.$refs.menuBar.hideContent()
			},
			onProgressChange: function(progress) {
				const percentage = progress / 100;
				const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
				this.rendition.display(location)
			},
			setTheme: function(index) {
				this.themes.select(this.themeList[index].name)
				this.defaultTheme = index;
			},
			registerTheme: function() {
				this.themeList.forEach(theme => {
					this.themes.register(theme.name, theme.style)
				})
			},
			setFontSize: function(fontSize) {
				this.defaultFontSize = fontSize;
				if (this.themes) {
					this.themes.fontSize(fontSize + 'px');
				}
			},
			toggleTitle: function() {
				this.ifTitleShow = !this.ifTitleShow;
				if (!this.ifTitleShow) {
					this.$refs.menuBar.hideSetting();
				}
			},
			changePage: function() {
				// 获取当前的位置信息
				const currentLocation = this.rendition.currentLocation()
				// 获取当前位置的进度百分比
				this.changeProgress = this.locations.percentageFromCfi(currentLocation.start.cfi)
				// 转化为0-100
				this.changeProgress = Math.round(this.changeProgress * 100)
			},
			prevPage: function () {
				// rendition.prev
				if (this.rendition) {
					this.rendition.prev().then(() => {
						this.changePage()
					})
				}
			},
			nextPage: function () {
				// rendition.next
				if (this.rendition) {
					this.rendition.next().then(() => {
						this.changePage()
					})
				}
			},
			// 电子书的解析和渲染
			showEpub: function () {
				// 生成Ebook
				this.book = new Epub(DOWNLOAD_URL);
				// 生成Rendition
				this.rendition = this.book.renderTo('read', {
					width: window.innerWidth,
					height: window.innerHeight,
				})
				// 通过Rendition.display渲染电子书
				this.rendition.display()
				// 获取Theme对象
				this.themes = this.rendition.themes;
				// 设置默认字体
				this.setFontSize(this.defaultFontSize);
				// this.themes.register(name, styles)
				// this.themes.select(name)
				this.registerTheme()
				this.setTheme(this.defaultTheme)
				// 获取Locations对象
				// 通过epubjs的钩子函数来实现
				this.book.ready.then(() => {
					this.navigation = this.book.navigation;
					return this.book.locations.generate()
				}).then(() => {
					this.locations = this.book.locations;
					this.bookAvailable = true;
				})
			}
		},
		mounted: function () {
			this.showEpub()
		}
	}
</script>

<style lang="scss" scoped>
	@import 'assets/styles/global';
	.ebook {
		position: relative;
		.read-wrapper {
			.mask {
				position: absolute;
				top: 0;
				left: 0;
				z-index: 100;
				display: flex;
				width: 100%;
				height: 100%;
				.left {
					flex: 0 0 px2rem(100);
				}
				.center {
					flex: 1;
				}
				.right {
					flex: 0 0 px2rem(100);
				}
			}
		}
	}
</style>