<template>
	<div class="v-box">
		<section class="box-content">
			<h2 class="path">
				<span class="p-s-f">{{txt1}}</span>
				<span>{{category.name}}</span>
			</h2>
			<div class="f-l">
				<VCategoryFrame :category="category"></VCategoryFrame>
			</div>
			<div class="f-r">
				<div class="content">
					<div class="block" :key="site.id" v-for="(site, index) in sites" @click="open(site)">
						<div class="name">
							<div class="i">
								<img v-lazy="site.iconLazyObj"/>
							</div>
							<span>{{site.name | clip(22)}}</span>
						</div>
						<div class="description" :title="site.description">
							<span>{{site.description | clip(60)}}</span>
						</div>
						<div class="heart" :title="likeTxt" @click.stop="liked(site, index)" >
							<span :style="site.liked&&`backgroundPositionX:-30px`" class="like"></span>
							<span class="text" v-if="!site.liked">{{likeTxt}}</span>
							<span class="liked-text" v-else>{{likedTxt}}</span>
						</div>
						<div ref="rAlert" class="alert" v-show="site.likedAlert">
							<span>{{alertTxt}}</span>
						</div>
					</div>
				</div>
			</div>
		</section>
	</div>
</template>

<script>
	import { likes } from 'mock/likes'
	export default {
		name: 'VBox',
		data() {
			return {
				category: {},
				sites: [],
				txt1: this.$txt.TXT_13,
				likeTxt: this.$txt.TXT_14,
				likedTxt: this.$txt.TXT_15,
				alertTxt: this.$txt.TXT_16,
			}
		},
		computed: {
			categoryid() {
				return this.$route.query.categoryid || ''
			},
		},
		mounted() {
			this.init()
			this.$nextTick(() => {
				this.category = _.cloneDeep(this.category)
			})
		},
		methods: {
			async init() {
				if (this.categoryid === '0099') { //喜欢的网单
					this.category = likes[0]
					this.sites = await this.getSite()
					this.sites = _.reverse(_.filter(this.sites, site => site.liked))
				} else {
					this.category = await this.constructCategory(this.categoryid)
					await this.construct()
				}
				this.sites && this.sites.forEach((site) => {
					site.iconLazyObj = {
						src: this.addHttp(site.icon),
						error: 'static/img/default-icon.png',
						loading: 'static/img/default-icon.png'
					}
				})
				this.category = _.cloneDeep(this.category)
				this.$api.reportByInfoc('liebao_urlchoose_detail:352 action:byte name:string url:string ver:byte', {
					action: 1,
					name: this.category.name,
					url: ''
				})
			},
			async construct() {
				const localSites = await this.getSite()
				this.sites = _.cloneDeep(this.category.sites)
				if (_.isEmpty(this.sites)) return
				for (let i = 0; i < this.sites.length; i++) {
					let site = this.sites[i],
						si = _.find(localSites, {'id': site.id + ''})
					!_.isEmpty(si) && (site.liked = si.liked, site.views = si.views)
					site.liked && (this.sites.splice(i, 1), this.sites.unshift(site))
				}
			},
			liked(site, i) {
				if (!site) return
				site.alertTimeout && (clearTimeout(site.alertTimeout))
				site.likedAlert = site.liked = !site.liked
				site.alertEl = this.$refs.rAlert[i]
				site.liked && (site.alertTimeout = setTimeout(() => {
					site.likedAlert = false
					const alert = site.alertEl
					this.$velocity(alert, {opacity: 0}, {
						duration: 400, complete: () => {
							alert.style.display = 'none', alert.style.opacity = 0.95
						}
					})
				}, 1000))
				this.saveSite(_.cloneDeep(site), this.categoryid)
				this.category = _.cloneDeep(this.category)
				site.liked && this.$api.reportByInfoc('liebao_urlchoose_detail:352 action:byte name:string url:string ver:byte', {
					action: 3,
					name: this.category.name,
					url: site.url
				})
			},
			open(site) {
				const url = site.href_url ? site.href_url : this.addHttp(site.url)
//				site.views = site.views? site.views+1 : 1
//				this.saveSite(_.cloneDeep(site), this.categoryid)
				window.open(url)
				this.$api.reportByInfoc('liebao_urlchoose_detail:352 action:byte name:string url:string ver:byte', {
					action: 2,
					name: this.category.name,
					url: site.url
				})
			},
		},
	}
</script>

<style lang="stylus" scoped>
	.box-content
		display flex
		margin 120px auto
		width 1043px
		.path
			position absolute
			font-size 12px
			color #5454a6
			.p-s-f
				&:after
					content: '>'
					margin-left 4px
		.f-l
			position relative
			top 35px
			width 252px
		.f-r
			position relative
			top 35px
			left 40px
			h2
				font-size 18px
				color #5454a6
				height 40px
			.content
				display flex
				flex-wrap wrap
				.block
					position relative
					margin 0 20px 20px 0
					border 1px solid #fff
					box-sizing border-box
					transition all .2s linear
					background #fff
					font-size 12px
					cursor pointer
					width 240px
					height 122px
					&:hover
						box-shadow 0 1px 20px -5px rgb(84, 84, 166)
					.name
						padding 10px 0 0 16px
						font-size 14px
						.i
							img
								position relative
								float left
								padding 6px 10px 0 0
								width 16px
								height 16px
						span
							line-height 2
							color #333333
					.description
						margin 3px 20px 0 42px
						span
							display block
							overflow hidden
							color #5b5b5b
							height 38px
					.heart
						position absolute
						bottom 6px
						right 10px
						cursor pointer
						font-size 14px
						z-index 0
						&:hover
							.like
								background-position -15px
						&:active
							.like
								background-position -30px
						.like
							top 4px
							right 8px
							float left
							background url("../../../../../assets/img/relation/like.png") no-repeat
							width 15px
							height 15px
						span
							position relative
							width 20px
							z-index 99
							cursor pointer
							&:nth-child(2)
								font-size 12px
						.text
							color #5454a6
						.liked-text
							color #ff3f5f
				.alert
					position absolute
					left 0
					top 0
					border-radius 4px
					width 100%
					height 100%
					background-color #fe5976
					opacity 0.95
					font-size 16px
					text-align center
					color #ffffff
					z-index 99
					span
						position absolute
						top 0
						left 0
						right 0
						bottom 0
						margin auto
						height 25px
</style>