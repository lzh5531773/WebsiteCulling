<template>
	<div class="v-category-frame">
		<div class="introduce">
			<div class="avatar">
				<img v-lazy="addHttp(category.avatar)"/>
				<span>{{category.name}}</span>
				<div class="mask"></div>
			</div>
			<div class="title">
				<a target="_blank" :title="category.name">{{category.name | clip(13)}}</a>
			</div>
			<div class="collect-btn">
				<VCollect :category="category"></VCollect>
			</div>
		</div>
	</div>
</template>

<script>
	import { resourceMock } from 'mock/resource'
	export default {
		name: 'VCategoryFrame',
		props: {
			category: {
				type: Object
			}
		}
	}
</script>

<style lang="stylus" scoped>
	.v-category-frame
		.introduce
			position relative
			float left
			border 1px solid #eae9ef
			-webkit-box-sizing border-box
			box-sizing border-box
			list-style none
			width 242px
			height 310px
			text-align center
			color gray
			-webkit-box-shadow 0 8px 18px rgba(0,0,0,.06)
			box-shadow 0 8px 18px rgba(0,0,0,.06)
			background #fff
			font-size 14px
			.avatar
				position relative
				overflow hidden
				height 160px
				-webkit-box-sizing border-box
				box-sizing border-box
				font-size 24px
				color white
				letter-spacing 7px
				img
					position relative
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
					overflow hidden
					float left
					text-align left
					width 102px
					color #333
			.collect-btn
				position relative
				top 38px
				right 14px
</style>