<template>
  <div class="landing-page">

    <div class="tag-list">
      <div class="tag-list_wrapper">
        <label id="allItems">Recent Items
          <input  type="radio" name="filterValue" value="all" v-model="valueToFilterBy">
        </label>
      </div>
      <div class="tag-list_wrapper" v-for="tag, key, index in completeTagList">
        <label>{{tag.tag}}
          <input type="radio" :id="tag.item_id" :name="tag.item_id" :value="tag.tag" v-model="valueToFilterBy">
        </label>
      </div>
    </div>

    <transition-group name="list-complete" tag="ul" mode="out-in" appear>
      <pocket-item v-for="(item, key, index) in filterByTag"
        @articleUrlSelected="launchArticleView"
        :item="item"
        :key="item.item_id"
        :index="index">
      </pocket-item>
    </transition-group>

  </div>
</template>

<script>
  const electron = require('electron')
  const {BrowserWindow, screen, webContents, dialog, ipcRenderer} = require('electron').remote
  const path = require('path')
  import {store} from '../SharedStore'
  import PocketItem from './LandingPageView/PocketItem'
  import _ from 'lodash'

  export default {
    components: {
      PocketItem,
    },

    data () {
      return {
        sharedState: store.state,
        valueToFilterBy: 'all'
      }
    },

    created () {
      this.getList()
      this.$electron.ipcRenderer.on('factorial-computed', function (event, url) {
        store.addArticle(url)
      })
      store.fetchFullList()
    },

    computed: {
      uniqueTagList: function () {
        var vm = this
        let filteredList =  _.filter(vm.sharedState.pocketList, 'tags')

        filteredList =  _.mapValues(filteredList, 'tags')
        filteredList = _.map(filteredList, function(obj) {
          return _.valuesIn(obj)
        })
        return _.uniqBy(_.flattenDeep(filteredList), 'tag')
      },

      completeTagList: function () {
        var vm = this
        let filteredList =  _.filter(vm.sharedState.fullList, 'tags')

        filteredList =  _.mapValues(filteredList, 'tags')
        filteredList = _.map(filteredList, function(obj) {
          return _.valuesIn(obj)
        })

        filteredList = _.uniqBy(_.flattenDeep(filteredList), 'tag')
        return _.orderBy(filteredList, ['tag'], ['asc'])

      },

      filterByTag: function () {
        var pocketList = _.orderBy(this.sharedState.pocketList, ['sort_id'], ['asc'])
        var filterByThis = this.valueToFilterBy
        var tagFetchResult = _.orderBy(this.sharedState.tagFetchResult, ['sort_id'], ['asc'])
        
        if (filterByThis !== 'all') {
          return this.sharedState.tagFetchResult
          // var filterItem = 'tags.' + filterByThis
          // return _.filter(pocketList, function(item) {
          //   return _.has(item, filterItem);
          // })
        } else {
          return pocketList
        }
      }
    },

    watch: {
      valueToFilterBy: 'filterByTagFetch'
    },

    methods: {
      filterByTagFetch: function () {
        store.fetchForTag(this.valueToFilterBy)
      },

      addArticle: function () {
        store.addArticle()
      },

      windowCommunication: function () {
        const windowID = BrowserWindow.getFocusedWindow().id
        mainWindow.webContents.on('did-finish-load', function () {
          const input = 100
          mainWindow.webContents.send('compute-factorial', input, windowID)
        })
      },

      launchArticleView: function (url) {
        this.$electron.ipcRenderer.send('openArticleWindow', url);
      },

      getFocusedWebContents: function () {
        let contents = webContents.getBrowserWindowOptions()
        console.log(contents.getURL())
      },

      getCurrentWebContents: function () {
        let webContents = this.$electron.remote.getCurrentWebContents();
        console.log(webContents);
        let allWindows = BrowserWindow.getAllWindows()
        console.log('Windows: ', allWindows);
      },

      getRequestToken: function () {
        store.runNodeAuth()
      },

      getAccessToken: function () {
        store.runGetAccessToken()
      },

      getList: function () {
        store.fetchPostsFromUserToken()
      },

      getFullList: function () {
        store.fetchFullList()
      }
    },

    name: 'landing-page'
  }
</script>

<style lang="scss">

#allItems {
  font-weight: 900;
  text-transform: uppercase;
  font-size: 16px;
}
.flip-list-move {
  transition: transform 1s;
}

.list-complete-item {
  transition: all 1.3s ease-out;
  margin: 1em;
  width: 200px;
}

.list-complete-enter {
    position: absolute;
}
.list-complete-enter, .list-complete-leave-active {
  opacity: 0;
  transform: translateY(-100%);
  transition: all .6s ease-in-out;
}
.list-complete-leave-active {
  position: absolute;
  // transition: all .6s ease;
    transform: translateY(200%);
  opacity: 0;
}


  .md-switch, .cursor {
    cursor: pointer;
  }
  .list, ul {
    display: inline-flex;
    left: 200px;
    flex-direction: row;
    flex-wrap: wrap;
    list-style-type: none;
    justify-content: space-around;
    position: fixed;
    right: 0;
    height: 100%;
    overflow-y: overlay;
    top: 64px;
    padding-bottom: 6.5em;
    // background: #4a4a4a;
    // background: rgba(74, 74, 74, 0.54);
    background: #6a6a6a;
    padding-top: 2em;
    overflow-x: hidden;

    &::-webkit-scrollbar {
      width: 10px;
    }

    &::-webkit-scrollbar-thumb {
      background-color: rgb(176, 176, 176)
    }

    &::-webkit-scrollbar-track {
      background: #404040;
    }

    &::-webkit-scrollbar-button {
      // background: #404040;
      height: 50px;
      border-radius: 50%;
    }

  }

  .article {
    &__wrapper {
      position: fixed;
      top: 65px;
      top: 50px;
      right: 0;
      width: 40%;
      bottom: 0;
      background-color: #fdfdfd;
      height: 100%;
      overflow-y: auto;
      padding: 1em .5em;
      left: 0;
      right: 0;
      width: 100vw;
    }
  }

  .tag-list {
    display: inline-flex;
    top: -33px;
    flex-direction: column;
    position: fixed;
    width: 200px;
    left: 0;
    overflow-y: auto;
    top: 0;
    height: 100%;
    padding: 6em 1em 2em 1em;

    & label {
      color: #3a3a3a;
      font-size: 15px;
      font-family: "Roboto";
      letter-spacing: .5px;
      line-height: 1.7;
      text-transform: capitalize;
      margin-bottom: -1em;
      cursor: pointer;
      font-weight: 400;
      font-family: -apple-system, BlinkMacSystemFont, sans-serif;
      // font-size: 17px;
      font-weight: 600;
      display: flex;
      justify-content: space-between;
      align-items: center;
      transition: color .15s ease;

      &:hover {
        cursor: pointer;
        color: #e91e63;
      }

      & input {
        cursor: pointer;
      }
    }

    &_wrapper {
      margin: 1em 0;
    }
  }

  .close-article {
    margin-bottom: 1em;
    position: fixed;
    top: 75px;
  }


</style>
