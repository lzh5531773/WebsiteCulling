<template>
	<div :class="{'from':!isFixed, 'from fixed': isFixed}" :style="{right:isFixed?right+'px':0}" v-if="sites&&sites.length>0">
		<div class="head">
			<h2>{{title}}</h2>
		</div>
		<section class="list-content">
			<div class="r-data" v-for="(data, i) in sites" :key="data.id" v-if="data.name">
				<a class="avatar" target="_blank" :href="data.href_url" :title="data.name" @click="open(data)">
					<img v-lazy="data.iconLazyObj"/>
				</a>
				<p class="title">
					<a target="_blank" class="name" :href="data.href_url" :title="data.name" @click="open(data)">{{data.name | clip(24)}}</a>
					<a target="_blank" class="description" :href="data.href_url" :title="data.description" @click="open(data)">{{data.description | clip(60)}}</a>
					<span class="r-t" @click="like(data)">
							<a class="like" :style="data.liked&&`backgroundPosition:-30px`"></a>
							<span>{{txt}}</span>
						</span>
				</p>
			</div>
		</section>
	</div>
</template>

<script>
	import { getHost, clipstring, getOperationFullTime } from 'utils/index'
	import { mockRelation } from 'mock/relation'
	export default {
		name: 'VOriginlist',
		data() {
			return {
				list: [],
				vm: [],
				title: this.$txt.TXT_34,
				txt: this.$txt.TXT_14,
				index: 0,
				screenWidth: document.body.clientWidth,
				right: document.body.clientWidth >= 1066 ? document.body.clientWidth - ((document.body.clientWidth - 1100) / 2 + 820 + 25 + 214) : document.body.clientWidth - (820 + 214)
			}
		},
		props: {
			isFixed: {
				type: Boolean,
				default: false
			},
			sites: {
				type: Array
			},
			categoryid: {}
		},
		watch: {
			screenWidth(val) {
				this.screenWidth = val
				this.right = this.screenWidth >= 1066 ? this.screenWidth - ((this.screenWidth - 1100) / 2 + 820 + 25 + 214) : this.screenWidth - (820 + 214)
			},
		},
		mounted() {
			const that = this
			window.onresize = () => {
				return (() => {
					window.screenWidth = document.body.clientWidth
					that.screenWidth = window.screenWidth
				})()
			}
		},
		methods: {
			open(data) {
				this.$api.reportByInfoc('liebao_urlchoose_detail:366 action:byte name:string url:string ver:byte action_time:string number1:int number2:int', {
					action: 2,
					name: this.categoryid + '',
					url: data.href_url,
					action_time: getOperationFullTime(new Date()),
					number1: 0,
					number2: 0
				})
			},
			like(data) {
				const store = {}
				store.id = data.id
				store.name = data.name
				store.url = data.url
				store.href_url = data.href_url
				store.description = data.description
				store.icon = data.icon
				store.liked = data.liked = !data.liked
				this.saveSite(store)
				store.liked && this.$api.reportByInfoc('liebao_urlchoose_detail:366 action:byte name:string url:string ver:byte action_time:string number1:int number2:int', {
					action: 3,
					name: this.categoryid + '',
					url: data.href_url,
					action_time: getOperationFullTime(new Date()),
					number1: 0,
					number2: 0
				})
			},
		},
	}
</script>

<style lang="stylus" scoped>
	.from
		position relative
		float left
		padding-bottom 20px
		width 214px
		height 82.6%
		@media screen and (max-height 1800px)
			height 86.6%
		@media screen and (max-height 800px)
			height 84.6%
		@media screen and (max-height 687px)
			height 82.6%
		.head
			border-bottom 1px solid #cdcdde
			color #6346de
			h2
				position relative
				bottom 5px
				font-size 14px
		.list-content
			position relative
			top 20px
			overflow auto
			height 100%
			&::-webkit-scrollbar
				display none
			.r-data
				height 75px
				font-size 14px
				.avatar
					float left
					img
						height 16px
						width 16px
				.title
					position relative
					bottom 3px
					overflow hidden
					padding-left 10px
					.name
						display block
						padding-bottom 3px
						overflow hidden
						text-overflow ellipsis
						white-space nowrap
						&:hover
							color #6346de
					a
						color #333333
					.description
						display block
						font-size 12px
						height 20px
						color #5b5b5b !important
						&:hover
							color #6346de!important
			.r-t
				display flex
				position absolute
				top 0
				right 0
				font-size 12px
				color #6346de
				cursor pointer
				&:hover
					.like
						background-position -15px
				&:active
					.like
						background-position -30px
				.like
					margin 2px 5px
					width 15px
					height 15px
					background url("../../../../assets/img/relation/like.png") no-repeat
	.fixed
		position fixed
		bottom 0
</style>