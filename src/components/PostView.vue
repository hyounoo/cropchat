<template>
  <form>
    <div>
      <div>
        <div>
          <div><img :src="this.catUrl"></div>
        </div>
      </div>
      <div>
        <div>
          <input id="username" v-model="title" type="text"/>
          <label for="username">Describe me</label>
        </div>
        <div class="actions">
          <a @click="postCat(catUrl, title)">
            POST A CAT
          </a>
        </div>
      </div>
    </div>
  </form>
</template>
<script>
import parse from 'xml-parser'
import postCat from '../mixins/postCat'
export default {
  mixins: [postCat],
  data() {
    return {
      catUrl: null,
      title: null
    }
  },
  mounted() {
    this.$http
      .get('https://thecatapi.com/api/images/get?format=xml&results_per_page=1')
      .then(response => {
        const catUrl = parse(response.body).root.children['0'].children['0']
          .children['0'].children['0'].content
        let img = new Image()
        img.onload = () => {
          this.catUrl = catUrl
        }
        img.src = catUrl
      })
  }
}
</script>
<style scoped>
.waiting {
  padding: 10px;
  color: #555;
}
</style>