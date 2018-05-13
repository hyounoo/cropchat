<template>
  <div>
    <div>
      <div></div>
      <div>
        <div v-for="picture in getCats()" :key="picture.id" @click="displayDetails(picture['.key'])">
          <div class="image-card__picture">
            <img :src="picture.url" />
          </div>
          <div>
            <span>{{ picture.comment }}</span>
          </div>
        </div>
      </div>
    </div>
    <router-link to="/post">
      <v-icon>add</v-icon>
    </router-link>
    <router-link class="take-picture-button" to="/camera">
      <v-icon>camera</v-icon>
    </router-link>
  </div>
</template>
<script>
export default {
  methods: {
    displayDetails(id) {
      this.$router.push({ name: 'detail', params: { id: id } })
    },
    getCats() {
      if (navigator.onLine) {
        this.saveCatsToCache()
        return this.$root.cat
      } else {
        return JSON.parse(localStorage.getItem('cats'))
      }
    },
    saveCatsToCache() {
      this.$root.$firebaseRefs.cat
        .orderByChild('created_at')
        .once('value', snapshot => {
          let cachedCats = []
          snapshot.forEach(catSnapshot => {
            let cachedCat = catSnapshot.val()
            cachedCat['.key'] = catSnapshot.key
            cachedCats.push(cachedCat)
          })
          localStorage.setItem('cats', JSON.stringify(cachedCats))
        })
    }
  },
  mounted() {
    this.saveCatsToCache()
  }
}
</script>
<style scoped>
.add-picture-button {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 998;
}
.image-card {
  position: relative;
  margin-bottom: 8px;
}
.image-card__picture > img {
  width: 100%;
}
.image-card__comment {
  position: absolute;
  bottom: 0;
  height: 52px;
  padding: 16px;
  text-align: right;
  background: rgba(0, 0, 0, 0.5);
}
.image-card__comment > span {
  color: #fff;
  font-size: 14px;
  font-weight: bold;
}
.take-picture-button {
  position: fixed;
  right: 24px;
  bottom: 90px;
  z-index: 5;
}
</style>