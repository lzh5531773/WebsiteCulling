<template>
  <div class="vue-recyclist">
    <div ref="list" class="vue-recyclist-items" :style="{height: height + 'px'}">
      <div v-for="(item, index) in visibleItems" class="vue-recyclist-item" :style="{transform: 'translate3d(0,' + item.top + 'px,0)'}">
        <div v-show="tombstone" :class="{'vue-recyclist-transition': tombstone}" :style="{opacity: +!item.loaded}">
          <slot name="tombstone"></slot>
        </div>
        <div :class="{'vue-recyclist-transition': tombstone}" :style="{opacity: +item.loaded}">
          <slot name="item" :data="item.data" :index="index" :lastIndex="visibleItems.length-1"></slot>
        </div>
      </div>
      <div class="vue-recyclist-pool">
        <div :ref="'item'+index" v-for="(item, index) in items" v-if="!item.tomb && !item.height"
          class="vue-recyclist-item vue-recyclist-invisible">
          <slot name="item" :data="item.data"></slot>
        </div>
        <div ref="tomb" class="vue-recyclist-item vue-recyclist-invisible">
          <slot name="tombstone"></slot>
        </div>
      </div>
    </div>
    <div v-show="spinner && !nomore && !tombstone"
         class="vue-recyclist-loading"
         :style="{visibility: loading ? 'visible' : 'hidden'}">
      <slot name="spinner">
        <div class="vue-recyclist-loading-content">
          <div class="cssloading-circle spinner"></div>
          <span>{{loadTxt}}</span>
        </div>
      </slot>
    </div>
    <div v-show="!loading && nomore" class="vue-recyclist-nomore">
      <slot name="nomore">
        <div></div>
      </slot>
    </div>
  </div>
</template>
<script>
  export default {
	  name: 'VRecyclist',
	  data() {
		  return {
			  items: [],
			  height: 0,
			  loadings: [],
			  start: 0,
			  startOffset: 0,
		  }
	  },
	  computed: {
		  visibleItems() {
			  return this.items.slice(Math.max(0, this.start - this.size), Math.min(this.items.length, this.start + this.size))
		  },
		  containerHeight() {
			  return this.scrollElement && this.scrollElement.offsetHeight || 0
		  },
		  tombHeight() {
			  return this.tombstone ? this.$refs.tomb && this.$refs.tomb.offsetHeight : 0
		  },
		  loading() {
			  return this.loadings.length
		  }
	  },
	  props: {
		  list: {
			  type: Array,
			  required: true
		  },
		  tombstone: {
			  type: Boolean,
			  default: false
		  },
		  size: {
			  type: Number,
			  default: 20
		  },
		  offset: {
			  type: Number,
			  default: 200
		  },
		  loadmore: {
			  type: Function,
			  required: true
		  },
		  spinner: {
			  type: Boolean,
			  default: true
		  },
		  nomore: {
			  type: Boolean,
			  default: false
		  },
		  scrollElement: {},
		  loadTxt: {
			  type: String
		  }
	  },
	  watch: {
		  list(arr) {
			  if (arr.length) {
				  this.loadings.pop()
				  if (!this.loading) {
					  this.loadItems()
				  }
			  } else {
				  this.init()
			  }
		  },
		  items(arr) {
			  if (arr.length > this.list.length) {
				  this.getItems()
			  }
		  },
		  scrollElement() {
			  this.scrollElement.addEventListener('scroll', this.onScroll.bind(this))
			  window.addEventListener('resize', this.onResize.bind(this))
			  this.init()
		  }
	  },
	  methods: {
		  init() {
			  this.reset()
			  this.load()
		  },
		  reset() {
			  this.items = []
			  this.height = this.top = this.start = 0
			  this.scrollElement.scrollTop = 0
		  },
		  load(flag) {
			  if (this.tombstone) {
				  this.items.length += this.size
				  this.loadItems()
			  } else if (!this.loading) {
				  this.getItems(flag)
			  }
		  },
		  getItems(flag) {
			  this.loadings.push(1)
			  this.loadmore(flag)
		  },
		  loadItems() {
			  let loads = []
			  let start = 0
			  let end = this.tombstone ? this.items.length : this.list.length
			  for (let i = start; i < end; i++) {
				  if (this.items[i] && this.items[i].loaded) {
					  continue
				  }
				  this.setItem(i, this.list[i] || null)
				  loads.push(this.$nextTick().then(() => {
					  this.updateItemHeight(i)
				  }))
			  }
			  Promise.all(loads).then(() => {
				  this.updateItemTop()
			  })
		  },
		  setItem(index, data) {
			  this.$set(this.items, index, {
				  data: data ? data : {},
				  height: 0,
				  top: -1000,
				  tomb: !data,
				  loaded: !!data
			  })
		  },
		  updateItemHeight(index) {
			  let cur = this.items[index]
			  let dom = this.$refs['item' + index]
			  if (dom && dom[0]) {
				  cur.height = dom[0].offsetHeight
			  } else {
				  cur.height = this.tombHeight
			  }
		  },
		  updateItemTop() {
			  this.height = 0
			  for (let i = 0; i < this.items.length; i++) {
				  let pre = this.items[i - 1]
				  this.items[i].top = pre ? pre.top + pre.height : 0
				  this.height += this.items[i].height
			  }
			  if (this.startOffset) {
				  this.setScrollTop()
			  }
			  this.updateIndex()
			  this.makeScrollable()
		  },
		  updateIndex() {
			  let top = this.scrollElement.scrollTop
			  for (let i = 0; i < this.items.length; i++) {
				  if (this.items[i].top > top) {
					  this.start = Math.max(0, i - 1)
					  break
				  }
			  }
		  },
		  getStartItemOffset() {
			  if (this.items[this.start]) {
				  this.startOffset = this.items[this.start].top - this.scrollElement.scrollTop
			  }
		  },
		  setScrollTop() {
			  if (this.items[this.start]) {
				  this.scrollElement.scrollTop = this.items[this.start].top - this.startOffset
				  this.startOffset = 0
			  }
		  },
		  makeScrollable() {
			  this.scrollElement.classList.add('vue-recyclist-scrollable')
		  },
		  onScroll() {
			  const listTop = this.scrollElement.children[1].offsetTop
			  if (this.scrollElement.scrollTop >= listTop) {
				  this.$emit('response', true)
			  } else {
				  this.$emit('response', false)
			  }
			  if (this.scrollElement.scrollTop + this.scrollElement.offsetHeight > this.height - this.offset) {
				  this.load(1)
			  }
			  this.updateIndex()
		  },
		  onResize() {
			  this.getStartItemOffset()
			  this.items.forEach((item) => {
				  item.loaded = false
			  })
			  this.loadItems()
		  }
	  },
	  destroyed() {
		  this.scrollElement && this.scrollElement.removeEventListener('scroll', this.onScroll.bind(this))
		  window.removeEventListener('resize', this.onResize.bind(this))
	  }
  }

</script>
<style src="../../../../assets/css/load.css"></style>
<style lang="stylus" rel="stylesheet/stylus">
  .vue-recyclist
    height 100%
    &.vue-recyclist-scrollable
      -webkit-overflow-scrolling touch
    .vue-recyclist-items
      position relative
      margin 0
      padding 0
      .vue-recyclist-invisible
        top -1000px
        visibility hidden
      .vue-recyclist-item
        position absolute
        width 100%
        .vue-recyclist-transition
          position absolute
          opacity 0
          transition-property opacity
          transition-duration 500ms
    .vue-recyclist-loading
      overflow hidden
      .vue-recyclist-loading-content
        position relative
        width 100%
        text-align center
        color #5b5b5b
        .spinner
          margin 10px auto
          width 20px
          height 20px
          right 20px
        span
          position absolute
          bottom 0
    .vue-recyclist-nomore
      overflow hidden
      margin 10px auto
      height 20px
      text-align center
      color #5b5b5b
</style>