<template>
  <div class="index-page">
    <div class="slider-wrapper">
      <slider>
        <div v-for="item in list" :key="item">
          <img width="100%" height="100%" :src="item" />
        </div>
      </slider>
    </div>
  </div>
</template>

<script>
  import Slider from 'base/slider/slider'

  export default {
    data() {
      return {
        list: ['http://y.gtimg.cn/music/photo_new/T003R720x288M000002DUw8O02w9ui.jpg', 'http://y.gtimg.cn/music/photo_new/T003R720x288M000001y2EOL0OT2UY.jpg', 'http://y.gtimg.cn/music/photo_new/T003R720x288M000001H04LT0tmRFg.jpg', 'http://y.gtimg.cn/music/photo_new/T003R720x288M000001zZjyc1zivBL.jpg', 'http://y.gtimg.cn/music/photo_new/T003R720x288M000002gJ9SZ4M9hMr.jpg']
      }
    },
    components: {
      Slider
    }
  }
</script>

<style scoped>
  .index-page {

  }
  .sslider-wrapper {
    width: 100%;
  }
</style>