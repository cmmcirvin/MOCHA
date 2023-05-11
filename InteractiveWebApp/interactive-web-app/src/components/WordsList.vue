<template>
  <div ref="div_wl" id="word_list_div">
    <li class="words_li" :id="'li_'+item" v-for="item in this.list_of_words" v-bind:key='item'>
      <div contenteditable>{{item}}</div>
      <Dropdown
        :id="'dwl_'+':'+item"
        class="dropdown_words_list"
        :options="categories"
        :disabled="false"
        :maxItem="categories.length"
        @selected="validateSelection('li_'+item)">
      </Dropdown>
    </li>
  </div>
</template>

<script>
import Dropdown from 'vue-simple-search-dropdown'
export default {
  /* eslint-disable */
  props: ['list_of_words', 'categories'],
  components: {
    Dropdown
  },
  computed: {
    word_list: function() {
      return this.list_of_words
    },
  },
  methods: {
    validateSelection(id) {
      let li = document.getElementById(id)
      if (li != null) {
        let isDescriptor = li.parentNode.parentNode.id == "confirmed_descriptors"
        if (!isDescriptor) {
          let descriptor_div = document.getElementById("confirmed_descriptors").childNodes[1]
          let cat = li.childNodes[2].childNodes[2].innerText.trim()
          let cats = [];
          for (let i = 0; i < this.categories.length; i++) {
            cats.push(this.categories[i].name);
          }
          if (cats.includes(cat)) {
            descriptor_div.appendChild(li)
          }
        }
      }
    }
  }
}
</script>

<style scoped>
li {
  font-size: 20px;
}
.words_li {
  display: flex;
  float: right;
}

.dropdown_words_list {
  padding-left: 15px;
}
</style>
