<template>
<div class="pure-g">
  <div class="text-box pure-u-1 pure-u-md-1 pure-u-lg-1 pure-u-xl-1">
    <div class="l-box">
      <h1 class="text-box-head">Photo Garally</h1>
      <p class="text-box-subhead">A collection of various photos from around the world</p>
    </div>
  </div>

  <div v-for="image in images" :key="image.photo_id" class="photo pure-u-1-3 pure-u-md-1-3 pure-u-lg-1-3 pure-u-xl-1-3">
    <router-link v-bind:to="{ name : 'photo', params : { photo_id: image.photo_id }}">
      <img v-bind:src="image_url_base + '/' + image.photo_id + '.' + image.type.split('/')[1]">
    </router-link>
  </div>

  <div class="pure-u-1 form-box" id="upload-image">
    <div class="l-box">
      <h2>Upload a Photo</h2>
      <input v-on:change="onFileChange" type="file" name="file" placeholder="Photo from your computer" accept="image/*" required>
      <button v-on:click="uploadImage" class="pure-button pure-button-primary">アップロード</button>
    </div>
  </div>
</div>
</template>

<script>
import axios from 'axios'
import appConfig from '../config'
import auth from '../auth'

const API_BASE_URL = appConfig.ApiBaseUrl
// const IMAGE_BASE_URL = appConfig.ImageBaseUrl

export default {
  data: function () {
    return {
      image_url_base: appConfig.ImageBaseUrl,
      uploadFile: null,
      images: []
    }
  },

  created: function () {
    this.listImages()
  },

  methods: {
    listImages: function () {
      var self = this
      var authHeader = auth.get_id_token()

      axios.get(API_BASE_URL + '/images/', {
        headers: {
          'Authorization': authHeader
        }
      }).then(function (res) {
        self.$data.images = res.data
      })
    },

    onFileChange: function (event) {
      this.uploadFile = event.target.files[0]
    },

    uploadImage: function () {
      var file = this.uploadFile
      var json = null
      var _this = this
      var authHeader = auth.get_id_token()

      // アップデート用の URL を取得
      var data = { size: file.size, type: file.type }
      axios
        .post(API_BASE_URL + '/images/', JSON.stringify(data), {
          headers: { Authorization: authHeader }
        })
        .then(function (res) {
          json = JSON.parse(JSON.stringify(res.data))

          axios
            .put(json['signed_url'], file, {
              headers: {
                'Content-Type': file.type
              }
            })
            .then(function (res) {
              json['status'] = 'Uploaded'
              axios.put(API_BASE_URL + '/images/', json, {
                headers: { 'Authorization': authHeader }
              }).then(function (res) {
                alert('Successfully uploaded photo.')
                _this.$router.go(_this.$router.currentRoute)
              })
            })
            .catch(function (error) {
              alert(error)
              console.log(error)
            })
        })
        .catch(function (error) {
          alert(error)
        })
    }
  }
}
</script>

<style>
.header .pure-menu {
  border-button-color: black;
  border-radius: 0;
}

.pure-menu-link {
  padding: 1em 0.7em;
}

.text-box-head {
  color: #fff;
  padding-button: 0.2em;
  font-weight: 400;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  sont-size: 24px;
}

.text-box-subhead {
  font-weight: normal;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1 {
  font-size: 2em;
  margin: 0.67em 0;
}

.l-box {
  padding: 2em;
}

.text-box {
  text-align: left;
  overflow: hidden;
  position: relative;
  height: 180px;
  background: rgb(49, 49, 49);
  color: rgb(255, 190, 94);
}

.photo {
  height: 250px;
  overflow: hidden;
}

.photo img {
  max-width: 100%;
  min-height: 250px;
  height: auto;
}
</style>
