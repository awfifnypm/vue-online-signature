<template>
  <div id="awfifnypm" style="width:200px;height:300px">
    <vueOnlineSignature ref="vueSignatureRef" :edg="270" :backIsCenter="true" domId="awfifnypm" :canvas-back="canvasBack"  :isCrop='true'/>
    <img v-if="imagesSRC" :src="imagesSRC" alt="" style="max-width: 100%">
    <span @click="onClick">生成</span>
  </div>
</template>

<script>
import vueOnlineSignature from '@/views/vueOnlineSignature'
export default {
  components: { vueOnlineSignature },
  data () {
    return {
      imagesSRC: '',
      canvasBack: require('@/assets/ceshi.png'),
      imgBack: require('@/assets/22.png')
    }
  },
  methods: {
    onClick () {
      this.$refs.vueSignatureRef.confirm()
        .then(res => {
          this.imagesSRC = res
        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div{
 height: 100vh
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
