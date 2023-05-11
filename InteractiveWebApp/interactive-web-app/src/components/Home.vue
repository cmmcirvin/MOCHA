<template>
  <div id="surrounding_container">
    <div id="deleted_words">
      Non descriptors
      <WordsList :list_of_words="non_descriptors" :categories="categories"/>
    </div>
    <div id="word_cloud">
      <div>Potential descriptors for {{words_for_cloud.length > 0 ? words_for_cloud[cloud_index][0] : "category"}}</div>
      <input id="word_list_file_uploader" v-on:change="word_list_uploaded" type="file" accept=".tab">
      <CustomCloud id="custom_cloud" v-if="words_for_cloud.length > 0" @updateDescriptors="updateDescriptors" :key="cloud_index" :data="words" :fontSizeMapper="fontSizeMapper" :width="750" :height="750" :font="'Roboto'" :padding="8"/>
      <button id="prev_page" v-if="words_for_cloud.length > 0" @click="prev_page">Prev</button>
      <button id="confirm_selections" v-if="words_for_cloud.length > 0" @click="confirmDescriptors">Confirm descriptor selections</button>
      <button id="clear_selections" v-if="words_for_cloud.length > 0" @click="clearDescriptors">Clear descriptor selections</button>
      <button id="generate_download_link" v-if="words_for_cloud.length > 0" @click="download_descriptors">Generate download link</button>
      <button id="next_page" v-if="words_for_cloud.length > 0" @click="next_page">Next</button>
      <PageNumber id="page_number_cloud" v-if="words_for_cloud.length > 0" :max_page="this.words_for_cloud.length" :curr_page="this.cloud_index"/>
    </div>
    <div id="confirmed_descriptors">
      New descriptors
    <WordsList :list_of_words="descriptors" :categories="categories"/>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import WordsList from './WordsList.vue'
import CustomCloud from './CustomCloud.vue'
import PageNumber from './PageNumber.vue'

export default {
  name: 'Home',
  components: {
    WordsList,
    CustomCloud,
    PageNumber,
},
  data() {
    return {
      words_by_category: {},
      words: [],
      descriptors: [],
      fontSizeMapper: word => Math.log2(word.value) * 5,
      non_descriptors: [],
      cloud_index: 0,
      max_words_in_cloud: 35,
      words_for_cloud: [],
      categories: [],
      to_download: [],
    }
  },
  methods: {
    download_descriptors() {
      let link_id = "download_descriptors_link"
      let prev_link = document.getElementById(link_id)
      if (!!prev_link) {
        prev_link.remove()
      }
      let link = document.createElement("a");
      link.id = link_id
      link.download = document.getElementById('word_list_file_uploader').files[0].name.slice(0, -4) + "_generated_descriptors.txt"
      link.innerHTML = "Download File"
      link.href = window.URL.createObjectURL(new Blob(this.to_download, {type:'text/plain'}))

      let cloud = document.getElementById("word_cloud")
      cloud.appendChild(link)
    },
    word_list_uploaded() {
      const reader = new FileReader()

      reader.onload = (e) => {
        let text = e.target.result
        text = text.split('\n')

        const categories = []
        for (let i = 1; i < text.length; i++) {
          if (text[i].split('\t')[1] && !categories.includes(text[i].split('\t')[1])) {
            categories[categories.length] = text[i].split('\t')[1]
          }
        }

        let words_by_category = {}
        for (let i = 0; i < categories.length; i++) {
          let category_words = []
          for (let j = 1; j < text.length; j++) {
            if (categories[i] == text[j].split('\t')[1] && text[j].split('\t')[2] === "not keyword\r") {
              category_words[category_words.length] = text[j].split('\t')[0]
            }
          }
          words_by_category[categories[i]] = category_words
        }

        let words_for_cloud = []
        for (const [category, values] of Object.entries(words_by_category)) {
          for (let i = 0; i < values.length; i += this.max_words_in_cloud) {
            let sliver = values.slice(i, i + this.max_words_in_cloud)
            for (let word_idx = 0; word_idx < sliver.length; word_idx++) {
              sliver[word_idx] = {text: sliver[word_idx].split("_").join(" "), value: Math.floor(Math.random() * 100 + 15)}
            }
            words_for_cloud.push([category, sliver])
          }
        }

        for (let i = 0; i < Object.keys(words_by_category).length; i++) {
          this.categories.push({ id: i + 1, name: Object.keys(words_by_category)[i]})
        }

        this.words_for_cloud = words_for_cloud
        this.words = words_for_cloud[0][1]
      }
      reader.readAsText(document.getElementById('word_list_file_uploader').files[0])
    },
    updateDescriptors(newDescriptor) {
      let cloud = document.getElementById("custom_cloud").childNodes[0].childNodes[0].childNodes
      for (let i = 0; i < cloud.length; i++) {
        if(cloud[i].__data__.text == newDescriptor) {
          if (this.descriptors.length > 0 && this.descriptors.includes(newDescriptor)) {
            this.descriptors.splice(this.descriptors.indexOf(newDescriptor), 1)
            cloud[i].style.fill = 'rgb(100, 100, 255)'
          }
          else {
              this.descriptors.splice(this.descriptors.length, 0, newDescriptor)
            cloud[i].style.fill = 'rgb(255, 0, 0)'
          }
        }
      }
    },
    confirmDescriptors() {

      // Add save functionality
      let new_descriptors = document.getElementById('confirmed_descriptors').childNodes[1].childNodes
      
      // Saves new descriptor names for alert, correct file saving
      let new_descriptor_names = []

      for (let i = 0; i < new_descriptors.length; i++) {
        // Gets new descriptor name
        let new_descriptor_name = new_descriptors[i].childNodes[0].innerText.trim()
        new_descriptor_names.push(new_descriptor_name)
      }

      // Confirmation request
      let confirmation_msg = `Are you sure you would like to mark selected words as descriptors?\nThis will discard all non-selected potential descriptors.\nSelected descriptors:\n${new_descriptor_names.join("\n")}`
      if (!confirm(confirmation_msg)) {
        return
      }

      for (let i = 0; i < new_descriptors.length; i++) {
        let name = new_descriptors[i].childNodes[0].innerText.trim()
        // Gets category from Dropdown
        let surrounding_div = new_descriptors[i].childNodes[2].children[1]
        let new_descriptor_category

        // Check if a specific option is selected to update category
        if (surrounding_div.childNodes.length > 1) {
          new_descriptor_category = this.words_for_cloud[this.cloud_index][0]
        }
        else {
          new_descriptor_category = new_descriptors[i].childNodes[2].children[1].children[0].innerText.trim()
        }

        this.to_download.push(`${name}\t${new_descriptor_category}\n`)
      }

      // update non descriptors
      for (let i = 0; i < this.words_for_cloud[this.cloud_index][1].length; i++) {
        if (!(this.descriptors.includes(this.words_for_cloud[this.cloud_index][1][i]["text"]))) {
          if (!(this.non_descriptors.includes(this.words_for_cloud[this.cloud_index][1][i]["text"]))) {
            this.non_descriptors.push(this.words_for_cloud[this.cloud_index][1][i]["text"])
          }
        }
      }

      this.cloud_index += 1
      this.words = this.words_for_cloud[this.cloud_index][1]

      document.getElementById("confirmed_descriptors").childNodes[1].innerHTML = ''

      this.descriptors.splice(0, this.descriptors.length)
    },
    clearDescriptors() {
      this.descriptors.splice(0, this.descriptors.length)
    },
    next_page() {
      this.cloud_index += 1
      this.words = this.words_for_cloud[this.cloud_index][1]
    },
    prev_page() {
      this.cloud_index -= 1
      this.words = this.words_for_cloud[this.cloud_index][1]
    }
  }
}
</script>

<style scoped>
h1, h2 {
  font-weight: normal;
}

#surrounding_container {
  position:relative;
  text-align:center;
  display:flex;
  width:100%;
  font-family:"Roboto Light";
  font-size:36px;
}

#deleted_words {
  display:inline-block;
  width:25%;
  text-align:center;
}

#word_cloud {
  display:block;
  width:50%;
  margin-left: auto;
  margin-right: auto;
  text-align:center;
}

#confirmed_descriptors {
  display:inline-block;
  width:25%;
  text-align:center;
}

#custom_cloud {
  display:block;
  
}

</style>
