<template>
  <section 
    class="autocomplete-container"
  >
    <input 
      :class="[{'active-input' : itemsList.length > 0},'input']" 
      @input="customWrite($event)"
      ref="selectedValue"
      placeholder="Search..."
      @keyup.up="upListItem"
      @keydown.down="downListItem"
      @keydown.enter="itemKeyboardSelected"
      @click-outside="ousideClick"
    />
    <!-- 
      think about $refs.selectedValue.select(), should improve 
      need to add lib for outside click handler
    -->
    <pre
      class="placeholder"
    >{{ autoSelected }}
    </pre>
    <ul 
      class="list"
    >
      <li 
        v-for="(item, index) in itemsList" 
        :class="[{ 'selected-item' : (selectedItemIndex == index)}, 'list-item']"
        :key="item.id"  
        @click="itemMouseSelected(index)"
      >
        <pre class="pre-in-list">{{ item[filterBy] }}</pre>
      </li>
    </ul>
  </section>
</template>
<script>

export default {
  name: 'AutoComplete',
  props: {
    items: Array,
    filterBy: String,
    addNewItem: Function
  },
  data() {
    return {
      query: '',
      selectedItemIndex: 0,
      autoSelected: '',
      itemsList: [],
      memo: this.memorizationItemsList()
    }
  },
  methods: {
    chnageInputValue() {
      this.$refs.selectedValue.value = this.itemsList[this.selectedItemIndex][this.filterBy]
      this.resetSelectedIndex()
      this.resetItemsList()
      this.resetQuery()
      this.resetPlaceholder()
    },
    itemKeyboardSelected() {
      if (this.itemsList.length === 0) return
      if (this.items
        .filter(e => e[this.filterBy] === this.query.trim())
        .length  === 0 ) {
          this.resetItemsList()
          this.resetPlaceholder()
          this.memo('ResetCacheAfterAddedNewItemInList')
          return this.addNewItem(this.query)
      }
      this.chnageInputValue()
    },
    itemMouseSelected(index) {
      if (this.itemsList.length === 0) return
      this.selectedItemIndex = index
      if (this.items
        .filter(e => e[this.filterBy] === this.itemsList[this.selectedItemIndex][this.filterBy].trim())
        .length  === 0 ) {
          this.resetItemsList()
          this.resetPlaceholder()
          this.memo('ResetCacheAfterAddedNewItemInList')
          return this.addNewItem(this.query)
      }
      this.chnageInputValue()
    },
    upListItem() {
      if (this.itemsList.length <= 1) return
      if (this.selectedItemIndex > 0) {
        this.$refs.selectedValue.value = this.itemsList[this.selectedItemIndex - 1][this.filterBy]
        this.selectedItemIndex -= 1
        if(this.selectedItemIndex === 0  ) {
          this.updatePlaceholder(1) 
          return this.resetSelectedIndex()
        }     
        this.resetPlaceholder()
      } else {
        this.resetPlaceholder()
        this.$refs.selectedValue.value = this.itemsList[this.itemsList.length - 1][this.filterBy]
        this.selectedItemIndex = this.itemsList.length - 1
      }
    },
    downListItem() {
      if (this.itemsList.length <= 1) return
      if (this.selectedItemIndex >= this.itemsList.length - 1) {
        this.$refs.selectedValue.value = this.itemsList[0][this.filterBy]
        this.updatePlaceholder(1)
        this.resetSelectedIndex()
      } else {
        this.resetPlaceholder()
        this.$refs.selectedValue.value = this.itemsList[this.selectedItemIndex + 1][this.filterBy]
        this.selectedItemIndex += 1
      }
    },
    resetSelectedIndex() {
      return this.selectedItemIndex = 0
    },
    resetItemsList() {
      return this.itemsList = []
    },
    resetPlaceholder() {
      return this.autoSelected = ''
    },
    resetQuery() {
      return this.query = ''
    },
    updatePlaceholder(index) {
      if (this.itemsList.length === 0) {
        return this.resetPlaceholder()
      } else {        
        if (this.query === this.itemsList[index][this.filterBy].slice(0, this.query.length)) {
          return this.autoSelected = this.itemsList[index][this.filterBy]
        } else {
          return this.resetPlaceholder()
        }
      }
    },
    customWrite(event) {
      if (event.target.value.trim().length === 0) {
        this.resetQuery()
        this.resetPlaceholder()
        return this.resetItemsList()
      }
      this.query = event.target.value
      if (this.query === '') {
        return []
      }
      // think about memorization
      // this.itemsList = this.items
      // .filter(e => e[this.filterBy] !== query)
      // .filter(e => e[this.filterBy].slice(0, query.length) === query)
      // .sort((a,b) => a[this.filterBy].length - b[this.filterBy].length)
      // .slice(0,5)
      this.itemsList = this.memo(this.query).slice(0,5)
      if (this.selectedItemIndex >= this.itemsList.length) {
        this.resetSelectedIndex()
      }
      this.updatePlaceholder(this.selectedItemIndex)
      return this.itemsList.unshift({id: 0, currency: this.query})
    },
    memorizationItemsList() {
      let cache = {} 
      return (query) => {
        if (query ==='ResetCacheAfterAddedNewItemInList') cache = {}
        if (query in cache) {
          return cache[query]
        } else {
          let result = this.items
          .filter(e => e[this.filterBy] !== query)
          .filter(e => e[this.filterBy].slice(0, query.length) === query)
          .sort((a,b) => a[this.filterBy].length - b[this.filterBy].length)
          cache[query] = result
          return result          
        }
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.autocomplete-container {
  width: 100%;
  position: relative;
}
.input,
.active-input {
  position: absolute;
  width: 276px;
  height: 32px;
  padding: 8px 12px;
  left: calc(50% - 151px);
  background: transparent;
  border: 1px solid gray;
  outline: none;
  font-size: 24px;
  line-height: 32px;
  z-index: 5;
}
.active-input {
  border-bottom: none;
}
.input:hover {
  box-shadow: 0 0 10px lightgray;
}
.placeholder {
  position: absolute;
  left: calc(50% - 138px);
  top: 9px;
  width: 276px;
  height: 32px;
  margin: 0;
  text-align: start;
  font-size: 24px;
  line-height: 32px;
  opacity: 0.4;
  font-family: Arial;
}
.list {
  list-style: none;
  width: 300px;
  margin: auto;
  display: flex;
  flex-direction: column;
  padding: 0;
  box-shadow: 0 0 10px lightgrey;
  position: absolute;
  top: 49px;
  left: calc(50% - 150px);
}
.list-item,
.selected-item {
  width: 276px;
  height: 16px;
  padding: 8px 12px;
  font-size: 16px;
  line-height: 16px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: start;
}
.list-item:hover,
.selected-item {
  cursor: pointer;
  background-color: lightgray;
  font-weight: bold;
}
.pre-in-list {
  margin: 0;
  font-family: Arial;
  width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
