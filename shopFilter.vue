<template>
  <div class="filter-box">
    <div class="box-operate clearix">
      <div class="operate-title clearix">
        <h1 class="title">已選標籤</h1>
        <a href="javascript:void(0)"
           @click="clearFilter"
           v-show="selectList.length">清空</a>
      </div>
      <div class="operate-tags clearix">
        <a href="javascript:void(0)"
           v-for="(item,index) in selectList"
           :key="index"
           @click="clearSingleFilter(item)">{{item.name}}</a>
      </div>
    </div>
    <div class="box-info clearix">
      <div class="info-block clearix"
           v-for="(item,index) in filterArr"
           :key="index">
        <h1 class="title clearix">{{item.title}}</h1>
        <div class="block-tags clearix">
          <label :for="it.id"
                 v-for="(it,idx) in item.list"
                 :key="idx">
            <input type="checkbox"
                   :id="it.id"
                   :value="it.id"
                   v-model="selectModel"
                   @change="filterCheckChange(it)">
            <p>{{it.name}}</p>
          </label>
        </div>
        <a href="javascript:void(0)"
           class="more-btn"
           v-if="item.list.length > 5"
           @click="moreFilterClick(item)">更多</a>
      </div>
    </div>
    <div class="bg-mask"
         v-show="moreFilterShow">
      <div class="filter-pupop">
        <div class="pupop-title">
          <h1 class="title">{{currentItem.title}}</h1>
          <i class="icon-close"
             @click="moreFilterShow = false"></i>
        </div>
        <div class="pupop-search">
          <input type="text"
                 placeholder="輸入關鍵字"
                 v-model="keyword"
                 @input="pupopSearch">
        </div>
        <div class="pupop-tag"
             v-show="currentItem.type === 'area' && !this.keyword">
          <label :for="it.id"
                 v-for="(it,idx) in currentItem.tagList"
                 :key="idx">
            <input type="radio"
                   :id="it.id"
                   :value="it.id"
                   name="areaTag"
                   v-model="tagModel"
                   @change="tagChange(it,idx)">
            <span>{{it.name}}</span>
          </label>
        </div>
        <div class="pupop-list">
          <label :for="'pupop-' + it.id"
                 v-for="(it,idx) in  currentItem.type === 'area' ? currentItem.tagList[currentTagIndex].list : currentItem.list"
                 :key="idx">
            <input type="checkbox"
                   :id="'pupop-' + it.id"
                   :value="it.id"
                   v-model="pupopSelectModel">
            <p>{{it.name}}</p>
          </label>
        </div>
        <div class="gradient-box"
             v-show="currentItem.type === 'area' ? currentItem.tagList[currentTagIndex].list && currentItem.tagList[currentTagIndex].list.length > 6 * 7 : currentItem.list && currentItem.list.length > 6 * 7"></div>
        <div class="pupop-operate">
          <a href="javascript:void(0)"
             @click="handlePupopOk">確定</a>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { mapActions } from 'vuex'
export default {
  name: 'ShopFilter',
  props: {
    list: {
      type: Array,
      required: true
    }
  },
  watch: {
    list: {
      handler (val) {
        this.filterArr = val
      },
      immediate: true
    }
  },
  data () {
    return {
      // 隱顯
      moreFilterShow: false,
      // 已選列表
      selectList: [],
      // 篩選組合model
      selectModel: [],
      // 篩選列表
      filterArr: [],
      // 當前操作篩選項
      currentItem: { title: '', tagList: [] },
      // 當前操作篩選項初始值
      initCurrentItem: {},
      // 當前操作大區索引
      currentTagIndex: '',
      // 搜索關鍵字
      keyword: '',
      // 大區tag model
      tagModel: '',
      // 彈窗選中model
      pupopSelectModel: []
    }
  },
  methods: {
    ...mapActions('shopFilter', ['setSearchParam']),
    /**
    * @desc handle click more filter tag
    * @param {Object} item current filter tag list
    * @author pika
    */
    moreFilterClick (item) {
      this.currentItem = item
      this.initCurrentItem = { ...item }
      if (item.type === 'area') {
        this.currentTagIndex = 0
        this.tagModel = item.tagList[0].id
      }
      this.pupopSelectModel = this.selectModel
      this.keyword = ''
      this.moreFilterShow = true
    },
    /**
    * @desc area tag change
    * @param {Object} item currentTag
    * @param {Number} index current tag index
    * @author pika
    */
    tagChange (item, index) {
      this.currentTagIndex = index
    },
    /**
    * @desc single filter operate
    * @param {Object} item
    * @author pika
    */
    clearSingleFilter (item) {
      this.selectList = this.selectList.filter(ele => ele.id !== item.id)
      this.selectModel = this.selectModel.filter(id => id !== item.id)
      this.emitData()
    },
    /**
    * @desc clear filter params
    * @author pika
    */
    clearFilter () {
      this.selectList = []
      this.selectModel = []
      this.emitData()
    },
    /**
    * @desc checkbox change of left sign filter change
    * @param {Object} item current check item
    * @author pika
    */
    filterCheckChange (item) {
      if (this.selectModel.some(id => id === item.id)) {
        this.selectList.push(item)
      } else {
        this.selectList = this.selectList.filter(ele => ele.id !== item.id)
      }
      this.emitData()
    },
    /**
    * @desc sure btn to click
    * @author pika
    */
    handlePupopOk () {
      // compact selectKey by pupopSelectModel and selectModel to know is add or not
      if (this.pupopSelectModel.length >= this.selectModel.length) {
        const keysAdd = this.pupopSelectModel.filter(pId => !this.selectModel.some(id => id === pId))
        const arrAdd = this.filterArr.map(item => item.list.filter(it => keysAdd.some(id => id === it.id))).flat()
        this.selectModel = this.selectModel.concat(keysAdd)
        this.selectList = this.selectList.concat(arrAdd)
      } else {
        this.selectModel = this.selectModel.filter(id => this.pupopSelectModel.some(pId => pId === id))
        this.selectList = this.selectList.filter(item => this.pupopSelectModel.some(pId => pId === item.id))
      }
      this.moreFilterShow = false
      this.emitData()
    },
    /**
    * @desc pupop filter search
    * @author pika
    */
    pupopSearch () {
      if (this.keyword) {
        if (this.initCurrentItem.type === 'area') {
          const areaFilter = this.filterArr.filter(item => item.type === 'area')[0].list.filter(item => item.name.indexOf(this.keyword) > -1)
          const tagList = []
          for (let i = 0, len = this.initCurrentItem.tagList.length; i < len; i++) {
            tagList[i] = {
              list: areaFilter
            }
          }
          this.currentItem = { ...this.currentItem, tagList }
        } else {
          this.currentItem = { ...this.currentItem, list: this.initCurrentItem.list.filter(item => item.name.indexOf(this.keyword) > -1) }
        }
      } else {
        this.currentItem = this.initCurrentItem
      }
    },
    /**
    * @desc emit data to father component
    * @return {Array} list
    * @author pika
    */
    async emitData () {
      const list = this.selectList.map(item => ({ id: String(item.id).replace('area-', ''), name: item.name, type: item.type }))
      const param = {
        type: list.filter(item => item.type === 'cuisine').map(tag => tag.id).join(',') || undefined,
        spend: list.filter(item => item.type === 'price').map(tag => tag.id).join(',') || undefined,
        business_region_id: list.filter(item => item.type === 'area').map(tag => tag.id).join(',') || undefined
      }
      await this.$store.dispatch('shopFilter/setSearchParam', param)
      this.$emit('sure', { list })
    }
  }
}
</script>
<style lang="scss" scoped>
.filter-box {
  background: #fff;
  width: $px * 240;
  padding: 0 $px * 20;
  overflow: hidden;
  box-sizing: border-box;
  h1 {
    &.title {
      color: #151517;
      font-size: $px * 16;
      line-height: $px * 22;
      font-weight: 600;
      padding: $px * 24 0 $px * 6 0;
      border-top: $px * 1 solid #f1f4f6;
    }
  }
  .box-operate {
    overflow: hidden;
    margin-bottom: $px * 18;
    .operate-title {
      display: flex;
      height: $px * 48;
      justify-content: space-between;
      align-items: center;
      overflow: hidden;
      h1 {
        &.title {
          padding: 0;
          border-top: 0;
        }
      }
      > a {
        font-size: $px * 16;
        line-height: $px * 22;
        display: inline-block;
        color: #151517;
      }
    }
    .operate-tags {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      > a {
        white-space: nowrap;
        word-break: keep-all;
        overflow: hidden;
        text-overflow: ellipsis;
        position: relative;
        display: inline-block;
        text-align: center;
        background: #ffe9e9;
        border: $px * 1 solid rgb(255, 54, 94);
        border-radius: $px * 8;
        color: #ff365e;
        font-size: $px * 14;
        height: $px * 36;
        line-height: $px * 36;
        padding: 0 $px * 40 0 $px * 20;
        margin: $px * 6 0;
        &::after {
          content: " ";
          position: absolute;
          top: $px * 11;
          right: $px * 13;
          display: inline-block;
          width: $px * 14;
          height: $px * 14;
          background: rgb(255, 54, 94);
        }
      }
    }
  }
  .box-info {
    font-size: 0;
    line-height: $px * 24;
    color: #151517;
    .block-tags {
      margin-bottom: $px * 18;
      max-height: $px * 150;
      overflow: hidden;
      > label {
        display: block;
        margin: $px * 6 0;
        > input[type="checkbox"] {
          display: none;
          &:checked {
            + p {
              color: #ff0059;
              overflow: hidden;
              text-overflow: ellipsis;
              &::before {
                background: #ff365e;
                border: 0;
              }
            }
          }
        }
        > p {
          font-size: $px * 14;
          position: relative;
          padding-left: $px * 36;
          &::before {
            content: " ";
            position: absolute;
            left: 0;
            top: 0;
            width: $px * 24;
            height: $px * 24;
            box-sizing: border-box;
            border: $px * 1 solid #999;
            border-radius: $px * 4;
            background: #fff;
          }
        }
      }
    }
    .more-btn {
      display: block;
      position: relative;
      font-size: $px * 16;
      color: #338fff;
      line-height: $px * 22;
      margin-bottom: $px * 18;
      &::after {
        content: " ";
        position: absolute;
        left: $px * 36;
        top: $px * 8;
        border-style: solid;
        border-color: #338fff transparent transparent transparent;
        border-width: $px * 7 $px * 7 0 $px * 7;
      }
    }
  }
}
.bg-mask {
  position: fixed;
  z-index: 99;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.6);
}
.filter-pupop {
  background: #fff;
  width: $px * 1000;
  box-sizing: border-box;
  position: relative;
  * {
    box-sizing: border-box;
  }
  .pupop-title {
    display: flex;
    height: $px * 80;
    padding: 0 $px * 20;
    justify-content: space-between;
    align-items: center;
    border-bottom: $px * 1 solid #979797;
    h1 {
      &.title {
        color: #151517;
        font-weight: 600;
        font-size: $px * 24;
        line-height: $px * 33;
        padding: 0;
        border: 0;
      }
    }
    .icon-close {
      display: inline-block;
      width: $px * 24;
      height: $px * 24;
      background: #000;
      background-size: 100%;
    }
  }
  .pupop-search {
    margin: $px * 24 0 $px * 38 0;
    padding: 0 $px * 20;
    > input {
      box-sizing: border-box;
      width: 100%;
      height: $px * 48;
      line-height: $px * 48;
      padding: 0 $px * 24 0 $px * 20;
      font-size: $px * 18;
      color: #999;
      background: url(~assets/img/i-search.png) no-repeat right #f4f4f4;
      background-size: $px * 24 $px * 24;
    }
  }
  .pupop-tag {
    white-space: nowrap;
    word-break: keep-all;
    overflow: auto;
    padding: 0 $px * 20;
    margin: 0 0 $px * 24 0;
    border-bottom: $px * 1 solid #979797;
    &::-webkit-scrollbar {
      display: none;
    }
    label {
      input[type="radio"] {
        display: none;
        &:checked {
          + span {
            background: #ff365e;
            color: #fff;
            border-radius: $px * 8;
          }
        }
        + span {
          padding: $px * 8 $px * 20;
          display: inline-block;
          box-sizing: border-box;
          line-height: $px * 24;
          font-size: $px * 18;
          margin: 0 $px * 80 $px * 12 0;
          color: #151517;
        }
      }
    }
  }
  .pupop-list {
    overflow: auto;
    padding: 0 $px * 20;
    border-bottom: $px * 1 solid #979797;
    height: $px * 48 * 7;
    &::-webkit-scrollbar {
      display: none;
    }
    label {
      float: left;
      width: $px * 150;
      margin: 0 $px * 12 $px * 24 0;
      height: $px * 24;
      line-height: $px * 24;
      font-size: 0;
      &:nth-child(6n) {
        margin-right: 0;
      }
      > input[type="checkbox"] {
        display: none;
        &:checked {
          + p {
            color: #ff0059;
            &:before {
              background: #ff365e;
              border: 0;
            }
          }
        }
        + p {
          font-size: $px * 14;
          position: relative;
          padding-left: $px * 36;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          word-break: keep-all;
          &::before {
            content: " ";
            position: absolute;
            left: 0;
            top: 0;
            width: $px * 24;
            height: $px * 24;
            box-sizing: border-box;
            border: $px * 1 solid #999;
            border-radius: $px * 4;
            background: #fff;
          }
        }
      }
    }
  }
  .gradient-box {
    position: absolute;
    z-index: 1;
    bottom: $px * 107;
    left: 0;
    width: 100%;
    height: $px * 82;
    background-image: linear-gradient(
      to top,
      rgb(255, 255, 255),
      rgba(255, 255, 255, 0.05),
      rgba(255, 255, 255, 0)
    );
  }
  .pupop-operate {
    display: flex;
    justify-content: center;
    align-items: center;
    height: $px * 106;
    > a {
      color: #fff;
      width: $px * 190;
      height: $px * 58;
      line-height: $px * 58;
      font-size: $px * 20;
      background: #ff365e;
      text-align: center;
      border-radius: $px * 8;
    }
  }
}
</style>
