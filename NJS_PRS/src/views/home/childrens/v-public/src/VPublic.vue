<template>
	<section class="public-page">
		<div class="containerRow">
			<ul>
				<li class="c-l" :key="data.id" v-for="(data, index) in dataList" v-if="data&&data.categories.length>0">
					<div class="head">
						<div class="left">
							<b class="flag"></b>
							<span class="head-t">
								{{data.name}}
							</span>
						</div>
						<a class="right" @click="pullEvent(data, index)" v-show="dataList.length > 2 && data.categories.length>3">
							<span v-if="data.event===0">{{pullTxt}}</span>
							<span v-else>{{pushTxt}}</span>
							<b ref="rPull" class="pull"></b>
						</a>
					</div>
					<div class="body" ref="rBody">
						<ul class="list">
							<li :key="category.id" v-for="category in data.showCategories" >
								<VItem :themeid="data.id" :category="category"></VItem>
							</li>
						</ul>
					</div>
				</li>
			</ul>
		</div>
		<VRecommend></VRecommend>
	</section>
</template>
<script>
	import { mapState } from 'vuex'
	import { getStore } from 'utils/index'
	import { themePath } from 'config/index'
	export default {
		name: 'VPublic',
		data() {
			return {
				dataList: [],
				pullTxt: this.$txt.TXT_25,
				pushTxt: this.$txt.TXT_26,
			}
		},
		mounted() {
			this.init()
		},
		computed:{
			...mapState([
				'position'
			])
		},
		watch: {

		},
		methods: {
			async init () {
				this.$api.reportByInfoc('liebao_urlchoose_find:355 action:byte value:byte hotsite:byte ver:byte url:string name:string',{action:1,value:0,hotsite:0,url:'',name:''})
				const store = getStore('THEME_IDS')? getStore('THEME_IDS'):''
				let themeid = ''
				if(store && store.length > 0) {
					themeid = store
				}else {
					let info = await this.getSelectedInfo()
					themeid = info? info:''
				}
				let themes = await this.fetch(themePath + themeid)
				this.dataList = _.cloneDeep(_.sortBy(_.forEach(themes, (d) => {
					if(themes.length > 2) {
						d.showCategories = d.categories? d.categories.slice(0, 3):[]
					}else {
						d.showCategories = _.cloneDeep(d.categories)
					}
					d.event = 0
				})), ['sort'])
				this.$nextTick(()=>{
					this.dataList.length <= 2 && this.$refs.rBody && this.$refs.rBody.forEach( (b, i) => {
						let count = this.dataList[i].categories.length
						b.style.maxHeight = 360*(Math.floor(count/3)+1)+'px'
					})
					this.position.name==='VPublic' && this.gotoPosition(this.position.scrolly)
				})
			},
			pullEvent(data, index){
				if(data.event===0){ //down
					data.showCategories = _.cloneDeep(data.categories)
					const durationVal = 120*(Math.floor(data.showCategories.length/3)+1)
					//下拉动画
					let count = this.dataList[index].categories.length
					this.$velocity(this.$refs.rPull[index], { rotateZ: "90deg" }, { duration: durationVal})
					this.$velocity(this.$refs.rBody[index], { maxHeight: 360*(Math.floor(count/3)+1) }, { duration: durationVal, complete: ()=>{
						data.event = 1
					}})
					this.$api.reportByInfoc('liebao_urlchoose_find:355 action:byte value:byte hotsite:byte ver:byte url:string name:string',{action:4,value:0,hotsite:0,url:'',name:this.dataList[index].id+''})
				}else if (data.event===1) { //up
					data.event = 0
					//收缩动画
					this.$velocity(this.$refs.rPull[index], { rotateZ: "0deg" })
					this.$velocity(this.$refs.rBody[index], { maxHeight: 365 }, {complete: () => {
						data.showCategories = _.cloneDeep(data.categories.slice(0,3))
					}})
				}
			},
			gotoPosition(position) {
				document.documentElement.scrollTop = position
				document.body.scrollTop = position
			}
		},
	}
</script>

<style lang="stylus" scoped>
	.public-page
		margin 112px auto
		width 1040px
		.containerRow
			float left
			margin 0 auto
			width 762px
			.c-l
				margin-bottom 10px
			.head
				position relative
				white-space nowrap
				height 24px
				.left
					float left
					.flag
						float left
						margin 3px 5px 0 0
						background url("../../../../../assets/img/home/f-f.png") no-repeat
						width 20px
						height 20px
					.head-t
						display inline-block
						float left
						vertical-align middle
						font-size 18px
						line-height 22px
						color #5454a6
				.right
					float right
					span
						font-size 12px
						color #333333
					.pull
						position relative
						top 5px
						float right
						width 16px
						height 16px
						cursor pointer
						background url("../../../../../assets/img/home/pull.png") no-repeat
			.body
				overflow hidden
				max-height 365px
				&:after
					content ''
					display block
					visibility hidden
					height 0
					clear both
					font-size 0
				.list
					padding-top 17px
					overflow hidden
					li
						position relative
						float left
						margin 0 20px 20px 0
						list-style none
						width 239px
						height 314px
						text-align center
						color gray
						-webkit-box-sizing border-box
						box-sizing border-box
						background #fff
						font-size 14px
						&:nth-child(3n)
							margin-right 0
</style>