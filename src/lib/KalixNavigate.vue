<template lang="pug">
  div.kalix-navigate(:class="{'small':menuChk}")
    ul.bd.bg(v-if="!menuChk")
      li(v-for="item in treeData" v-bind:key="item.id" v-bind:class="{'active':item.isShow}")
        div.s-flex.tit.tit-txt(@click="showTree(item,$event)")
          div.s-flex_item
            i.tit_icon(:class="bindClass(item.iconCls)")
            span.txt {{item.text}}
          div.arrow
            i(:class="showIcon(item.isShow)")
        el-collapse-transition
          div.mn(v-show="item.isShow")
            ul
              li(v-for="item in item.children" v-bind:key="item.id")
                div.tit(v-on:click="selectItem(item)" v-bind:class="currentCls(item)")
                  i.tit_icon(:class="bindClass(item.iconCls)")
                  | {{item.text}}
    ul.bd.samll(v-if="menuChk")
      li(v-for="item in treeData")
        div.s-flex.tit(v-bind:class="{'active':item.isShow}")
          i.tit_icon(:class="bindClass(item.iconCls)")
          div.mn
            div.txt {{item.text}}
            ul
              li.tit(v-for="item in item.children"
              v-bind:key="item.id"
              v-on:click="selectItem(item)")
                i.tit_icon(:class="bindClass(item.iconCls)")
                | {{item.text}}
</template>

<script type="text/ecmascript-6">
  import Cache from '../common/cache'

  export default {
    name: 'KalixNavigate',
    props: {
      cacheTime: {
        type: Number,
        default: ''
      },
      url: {
        type: String,
        default: ''
      },
      menuChk: {
        default: false
      }
    },
    data() {
      return {
        name: 'kalixNav',
        currApp: '',
        currFun: '',
        activeName: '1',
        obj: {'name': 'aa'},
        treeData: [],
        clickedNode: null,
        flag: true
      }
    },
    activated() {
      this.fetchData()
    },
    mounted() {
    },
    watch: {
      '$route'(to, from) {
        if (to.path !== '/' && to.path !== '/login') {
          this.fetchData()
        }
      }
    },
    methods: {
      fetchData() {
        this.treeData = []
        console.log('this.$route:', this.$route)
        console.log('this.$route.params.app:', this.$route.params.app)
        if (this.$route.name === 'login') {
          return
        }
        /*if (!appId) {
          console.log('999 ', appId)
          let toolListData = JSON.parse(Cache.get('toolListData'))
          if (!toolListData) {
            return
          }
          console.log('9990 ', appId)
          appId = toolListData[0].id
          console.log('9991 ', appId)
        }*/
        let d = new Date()
        let cd = d.getTime()
        let treeListData = {}
        this.currApp = this.$route.params.app
        this.currFun = this.$route.params.fun || ''
        if (Cache.get('treeListData')) {
          treeListData = JSON.parse(Cache.get('treeListData'))
        }
        if (treeListData.createDate && (treeListData.createDate - cd) < this.cacheTime && treeListData[this.currApp]) {
          this.treeData = treeListData[this.currApp]
          this.setItemShow()
        } else {
          const data = {_dc: cd, node: 'root'}
          if (this.flag) {
            this.flag = false
            this.$http.get(
              this.url + this.currApp,
              {
                params: data
              }).then(response => {
              this.flag = true
              let nowDate = new Date()
              if (response.data && response.data.code !== 401) {
                this.treeData = response.data
                if (this.treeData.length) {
                  this.treeData.forEach(function (e, i) {
                    // Vue.set(e, 'isShow', false)
                  })
                  treeListData[this.currApp] = this.treeData
                  treeListData.createDate = nowDate.getTime()
                  this.setItemShow()
                  Cache.save('treeListData', JSON.stringify(treeListData))
                }
              }
            })
          }
        }
      },
      setItemShow() {
//        let routeName = this.currApp + '/' + this.currFun
        let routeName = this.currFun.toLowerCase()
        this.treeData.forEach((item) => {
          item.isShow = false
          let temp = item.children.find(function (e) {
            let routeId = e.routeId.split('/').pop()
            return routeId.toLowerCase() === routeName
          })
          if (temp) {
            item.isShow = true
          }
        })
      },
      bindClass(e) {
        return e
      },
      showTree(e) {
        this.clickedNode = e
        this.treeData.forEach((item) => {
          if (item !== e) {
            item.isShow = false
          } else {
            item.isShow = !item.isShow
          }
        })
      },
      showIcon(e) {
        return e ? 'el-icon-caret-bottom' : 'el-icon-caret-right'
      },
      selectItem(item) {
        this.$router.push({path: `/${this.currApp}/${item.routeId.split('/').pop()}`})
        Cache.save('currentTreeListItem', JSON.stringify(item))
      },
      currentCls(item) {
        return item.routeId.split('/').pop().toLowerCase() === this.currFun.toLowerCase() ? 'active' : ''
      }
    },
    components: {}
  }
</script>

<style lang="stylus" type="text/stylus">
  .kalix-navigate
    width 250px
    height 100%
    text-align left
    overflow hidden
    transition width .2s
    z-index 999
    overflow-y auto
    ul
      li
        width 100%
        .tit
          height 64px
          line-height 64px
          cursor pointer
        .tit_icon
          margin-right 12px
          font-size 24px
      &.bg
        li
          .tit
            padding-left 22px
            padding-right 12px
            position relative
            transition color 0.5s
          .txt, .arrow
            transition opacity .2s
          .txt
            display inline-block
          .arrow
            width 12px
          .mn
            .tit
              &:hover
                background-color #3c4a57
                color #d6d9dc
              padding-left 44px
              &:before
                top 0
                left 0
                bottom 0
                width 5px
                //background-color #42b983
                content ''
                position absolute
                display block
                visibility hidden
              &.active
                background-color #384657
                color #ffffff
                &:before
                  visibility visible

    &.small
      width 65px
      overflow visible
      li
        .tit
          display block
          position relative
          text-align center
          &:before
            top 0
            left 0
            bottom 0
            width 5px
            content ''
            position absolute
            display block
            visibility hidden
          &.active
            &:before
              visibility visible
          .mn
            position absolute
            left 50px
            top 0
            width 180px
            text-align left
            display none
            ul
              li.tit
                padding-left 20px
                text-align left
                box-sizing border-box
          &:hover
            .mn
              display block

</style>