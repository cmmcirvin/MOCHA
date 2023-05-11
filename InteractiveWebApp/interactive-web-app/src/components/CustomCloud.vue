<script>
/* eslint-disable */
import Cloud from 'vue-d3-cloud'

export default {
    name: 'CustomCloud',
    extends: Cloud,
    mounted() {
        let cloud = document.getElementById("custom_cloud").childNodes[0].childNodes[0].childNodes
        for (let i = 0; i < cloud.length; i++) {
            cloud[i].style.fill = 'rgb(100,100,255)'
        }
    },
    methods: {
        onWordClick: function(word) {
            // let cloud = document.getElementById("custom_cloud").childNodes[0].childNodes[0].childNodes
            // for (let i = 0; i < cloud.length; i++) {
            //     if(cloud[i].__data__.text == word.text) {
            //         if (cloud[i].style.fill != 'rgb(255, 0, 0)'){
            //             cloud[i].style.fill = 'rgb(255, 0, 0)'
            //         }
            //         else {
            //             cloud[i].style.fill = 'rgb(100, 100, 255)'
            //         }
            //     }
            // }

            this.$emit('updateDescriptors', word.text)
        }
    }
}

</script>