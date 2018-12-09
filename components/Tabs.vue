<template>
  <div class="tabs-component">
    <ul ref="tabHeader" role="tablist" class="tabs-component-tabs">
      <li
        v-for="(tab, i) in tabs"
        :key="i"
        :class="{ 'is-active': tab.isActive, 'is-disabled': tab.isDisabled }"
        class="tabs-component-tab"
        role="presentation"
        v-show="tab.isVisible"
      >
        <a
          v-if="options.useUrlFragment" 
          v-html="tab.header"
          :aria-controls="tab.hash"
          :aria-selected="tab.isActive"
          @click="selectTab(tab.hash, $event)"
          :href="tab.hash"
          class="tabs-component-tab-a"
          role="tab"
        ></a>
        <span
          v-else
          v-html="tab.header"
          :aria-selected="tab.isActive"
          @click="selectTab(tab.hash, $event)"
          class="tabs-component-tab-a"
          role="tab"
        ></span>
      </li>
      <li class="tabs-component-tab-gap" :style="gapStyle"></li>
    </ul>
    <div class="tabs-component-panels">
      <slot/>
    </div>
  </div>
</template>

<script>
  // fork https://github.com/spatie/vue-tabs-component
  export default {
    props: {
      options: {
        type: Object,
        required: false,
        default: () => ({
          useUrlFragment: true,
          defaultTabHash: null
        })
      }
    },
    data: () => ({
      tabs: [],
      activeTabHash: '',
      activeTabIndex: 0,
      lastActiveTabHash: '',
      gapStyle: {
        'visiblity': false,
        'left': 0
      }
    }),
    created() {
      this.tabs = this.$children;
    },
    mounted() {
      if(this.options.defaultTabHash !== null && this.findTab("#" + this.options.defaultTabHash)) {
        this.selectTab("#" + this.options.defaultTabHash)
        return
      }
      if (this.tabs.length) {
        this.selectTab(this.tabs[0].hash)
      }
    },
    methods: {
      findTab(hash) {
        return this.tabs.find(tab => tab.hash === hash)
      },
      selectTab(selectedTabHash, event) {
        // See if we should store the hash in the url fragment.
        if (event && !this.options.useUrlFragment) {
            event.preventDefault()
        }
        const selectedTab = this.findTab(selectedTabHash)
        if (! selectedTab) {
            return
        }
        if (selectedTab.isDisabled) {
            event.preventDefault()
            return
        }
        if (this.lastActiveTabHash === selectedTab.hash) {
            this.$emit('clicked', { tab: selectedTab })
            return
        }
        this.tabs.forEach(tab => {
            tab.isActive = (tab.hash === selectedTab.hash)
        })

        setTimeout(() => {
          this.gapStyle = {
            visibility: 'visible',
            ...this.getElementOffsetLeft()
          }
        }, 0)

        this.$emit('changed', { tab: selectedTab })
        this.activeTabHash = selectedTab.hash
        this.activeTabIndex = this.getTabIndex(selectedTabHash)
        this.lastActiveTabHash = this.activeTabHash = selectedTab.hash
      },
      setTabVisible(hash, visible) {
          const tab = this.findTab(hash)
          if (! tab) {
              return
          }
          tab.isVisible = visible
          if (tab.isActive) {
            // If tab is active, set a different one as active.
            tab.isActive = visible
            this.tabs.every((tab, index, array) => {
              if (tab.isVisible) {
                tab.isActive = true
                return false
              }
              return true
            })
          }
      },
        
      getTabIndex(hash){
        const tab = this.findTab(hash)
        return this.tabs.indexOf(tab)
      },
        
      getTabHash(index){
        const tab = this.tabs.find(tab => this.tabs.indexOf(tab) === index)
          
        if (!tab) {
          return
        }
            
        return tab.hash
      },
        
      getActiveTab(){
        return this.findTab(this.activeTabHash)
      },
        
      getActiveTabIndex() {
        return this.getTabIndex(this.activeTabHash)
      },

      getElementOffsetLeft () {
        const tabs = this.$refs.tabHeader.children
        const activeTab = tabs[this.activeTabIndex]
        return {
          left: activeTab.offsetLeft + 'px',
          width: activeTab.offsetWidth + 'px'
        }
      }
    }
  }
</script>

<style lang="stylus">
.tabs-component
  margin 4em 0 0

.tabs-component-tabs
  position relative
  align-items: stretch
  display flex
  justify-content center
  margin 0
  padding 0

.tabs-component-tab
  color lighten($viceTextColor, 20%)
  font-size px2rem(16px)
  font-weight 600
  margin 0 px2rem(30px)
  list-style none
  cursor pointer
  &-gap
    position absolute
    top 100%
    left 0
    content ''
    width 5em
    height 3px
    list-style none
    background $accentColor
    visibility hidden
    transition all .3s

.tabs-component-tab:hover
.tabs-component-tab.is-active
  color $accentColor

.tabs-component-tab.is-disabled * 
  color #cdcdcd
  cursor not-allowed !important

.tabs-component-tab-a
  align-items center
  color inherit
  display flex
  padding px2rem(10px)
  text-decoration none

.tabs-component-panels
  padding 4em 2em
.tabs-component-panel
  display flex
  justify-content center
  align-items: start
  div
    &:only-child
      flex 1
      width auto
  div
    width 50%
</style>