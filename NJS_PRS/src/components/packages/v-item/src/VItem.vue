<template>
	<div class="item">
		<div class="avatar" :title="category.name" @click="open(1, 1)">
			<img v-lazy="addHttp(category.avatar)" :style="category.id===`0099`&&`position:relative;bottom:4px`"/>
			<span>{{category.name}}</span>
			<div class="mask"></div>
		</div>
		<p class="title">
			<a target="_blank" :title="category.name" @click="open(1, 1)">{{category.name | clip(13)}}</a>
			<span class="by">{{by}}</span>
		</p>
		<div class="site-list">
			<p v-if="index<=2" v-for="(site, index) in category.sites">
				<a :href="site.href_url" class="site-n" :title="site.name" target="_blank"  @click.stop="open(2, 2, site)">{{site.name | clip(13)}}</a>
				<a :href="site.href_url" class="site-u" :title="site.url" target="_blank"  @click.stop="open(2, 2, site)">{{site.url | clip(16)}}</a>
			</p>
		</div>
		<div class="more" @click="open(1, 3)">
			<span>{{moreTxt}}</span>
			<img src="../../../../assets/img/home/more.png"/>
		</div>
	</div>
</template>

<script>
	import { styleConfigPath } from 'config/index'
	import styleConfigs from '../../../../external/style'
	export default {
		name: 'VItem',
		data() {
			return {
				moreTxt: this.$txt.TXT_27,
			}
		},
		computed: {
			by() {
				return this.category.by ? 'by ' + this.category.by : ''
			}
		},
		props: {
			category: {
				type: Object
			},
			sort: {
				type: Number
			}
		},
		methods: {
			async open(flag, type, site) {
				if (flag === 1) {
					await this.distribute()
					if (this.category.id === '0099') {
						this.$api.reportByInfoc('liebao_urlchoose_mine:353 action:byte url:string value:byte ver:byte', {
							action: 7,
							url: '',
							value: 0
						})
						return
					}
					let categories = await this.getForm(),
						category = _.find(categories, {id: this.category.id + ''})
					category = _.isEmpty(category) ? this.category : category
					this.saveForm(category)
					!this.sort && this.$api.reportByInfoc('liebao_urlchoose_find:355 action:byte value:byte hotsite:byte ver:byte url:string name:string', {
						action: 2,
						value: type,
						hotsite: 0,
						url: '',
						name: this.category.id + ''
					})
					this.sort && this.sort === 3 && this.$api.reportByInfoc('liebao_urlchoose_mine:353 action:byte url:string value:byte ver:byte', {
						action: 5,
						url: '',
						value: this.category.id
					})
					this.sort && this.sort === 5 && type === 1 && this.$api.reportByInfoc('liebao_urlchoose_mine:353 action:byte url:string value:byte ver:byte', {
						action: 9,
						url: '',
						value: this.category.id
					})
				} else if (flag === 2 && site) {
					let sites = await this.getSite(),
						s = _.find(sites, {id: site.id + ''})
					s = _.isEmpty(s) ? site : s
					!this.sort && this.$api.reportByInfoc('liebao_urlchoose_find:355 action:byte value:byte hotsite:byte ver:byte url:string name:string', {
						action: 2,
						value: type,
						hotsite: 0,
						url: site.url,
						name: this.category.id + ''
					})
				}
			},
			async distribute() {
				let styles = await this.getJSON(styleConfigPath),
					path = 'collection'
				styles = !_.isEmpty(styles) ? styles : styleConfigs
				const style = _.find(styles, {categoryId: this.category.id}),
					data = {
						path: path
					};
				data.query = {
					categoryid: this.category.id,
					name: this.category.name,
					component: !_.isEmpty(style) ? style.component : 'VBox',
					config: !_.isEmpty(style) ? JSON.stringify(style.config) : ''
				}
				this.$router.push(data)
			}
		},
	}
</script>

<style lang="stylus" scoped>
	.item
		height 100%
		width 100%
		.avatar
			position relative
			overflow hidden
			height 51%
			-webkit-box-sizing border-box
			box-sizing border-box
			font-size 24px
			color white
			letter-spacing 7px
			cursor pointer
			span
				position absolute
				top 0
				right 0
				left 0
				bottom 0
				overflow hidden
				margin auto
				height 40px
				z-index 99
			.mask
				position absolute
				left 0
				top 0
				background-color black
				width 240px
				height 156px
				opacity 0.2
				z-index 9
		.title
			overflow hidden
			margin 16px 16px 7px
			color #333
			font-size 16px
			font-weight 600
			font-synthesis style
			text-overflow ellipsis
			white-space nowrap
			border-bottom 1px solid #949494
			height 30px
			.by
				float right
				font-size 12px
				color #949494
				line-height 2
			a
				position relative
				float left
				overflow hidden
				text-align left
				color #333
		.site-list
			margin 16px 16px 7px
			p
				overflow hidden
				height 20px
				font-size 12px
				color #333333
			.site-n
				float left
				text-align left
				color #333333
				&:hover
					color #5454a6
					border-bottom 1px solid #5454a6
			.site-u
				float right
				text-align right
				color #333
				&:hover
					color #5454a6
					border-bottom 1px solid #5454a6
		.more
			position absolute
			right 0
			bottom 0
			float right
			margin 6px 16px
			font-size 12px
			color #5454a6
			height 20px
			cursor pointer
			span
				float right
</style>