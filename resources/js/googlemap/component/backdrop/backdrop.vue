<template>
  <div class="backdrop" id="backdrop" v-on:click="closeBackdrop">
    <div class="control-area">
      <label class="control" v-on:click="uploadClick">
        Upload
        <input type="file" v-on:change="previewImage">
      </label>
      <div class="control" v-on:click="deletePhoto">Delete</div>
    </div>
    <div class="carousel-area">
      <!-- Loading spinner -->
      <i class="fa fa-spinner fa-spin" v-if="spinner"></i>

      <Carousel v-if="!spinner"/>
    </div>
  </div>
</template>

<script>
import carousel from '../carousel/carousel.vue';

export default {
  components: {
    'Carousel': carousel
  },
  data() {
    return {
    }
  },
  methods: {
    uploadClick: function () {
      let payload = {
        src: "",
        value: ""
      }
      this.$store.commit('photo/uploadPhoto', payload);
      this.$store.commit('photo/isUpload', true);
      this.$store.commit('carousel/position', 0);
      this.$store.commit('carousel/photo_index', 0);
    },
    deletePhoto: function () {
      this.$store.commit('map/spinner', true);
      let data = {
        location_id: this.markerId,
        photo_id: this.photoList[this.photoIndex].id
      }
      axios.post('/api/photo/delete', data).then((res) => {
        if (res.data.success) {
          this.$store.commit('photo/deletePhoto', this.photoIndex);
          if (this.photoList.length == 0) {
            this.$store.commit('carousel/initPosition');
            this.$store.commit('photo/isUpload', true);
          } else if (this.photoIndex == this.photoList.length) {
            this.$store.commit('carousel/deletePosition');
          }
          this.$store.commit('map/spinner', false);
        }
      }).catch((err) => { console.log(err) })
    },
    previewImage: function (event) {
      let payload = {};

      let reader = new FileReader();
      reader.readAsDataURL(event.target.files[0]);
      reader.onload = () => {
        payload.src = reader.result;
        payload.value = event.target.files[0];
        this.$store.commit('photo/uploadPhoto', payload);
      }
    },
    closeBackdrop: function (event) {
      if (event.target.id == "backdrop") {
        this.$store.commit('map/backdrop', false);
        this.$store.commit('photo/setPhotoList', []);
        this.$store.dispatch('carousel/backdropClose');

        let payload = {
          src: "",
          value: ""
        }
        this.$store.commit('photo/uploadPhoto', payload);
      }
    }
  },
  computed: {
    spinner: function () {
      return this.$store.state.map.spinner;
    },
    photoList: function () {
      return this.$store.state.photo.photo_list;
    },
    photoIndex: function () {
      return this.$store.state.carousel.photo_index;
    },
    markerId: function () {
      return this.$store.state.map.mark_id;
    },
    position: function () {
      return this.$store.state.carousel.position;
    }
  }
}
</script>

<style lang="scss" scoped>
@import "./backdrop.scss";
</style>
