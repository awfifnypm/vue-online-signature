<template>
  <div id="awfifnypm">
    <vueOnlineSignature ref="vueSignatureRef" v-bind="params"/>
    <img v-if="imagesSRC" :src="imagesSRC" alt="" style="max-width: 100%">
    <span @click="onClick">生成</span>
    <span @click="recover">恢复</span>
  </div>
</template>

<script>
// import vueOnlineSignature from 'vue-online-signature'
import vueOnlineSignature from './vueOnlineSignature.vue'
export default {
  components: { vueOnlineSignature },
  data () {
    return {
      imagesSRC: '',
      canvasBack: require('@/assets/images/ceshi.png'),
      imgBack: require('@/assets/images/22.png'),
      params: {
        width: 0,
        height: 0,
        lineWidth: 5,
        lineColor: '#000',
        canvasBack: require('@/assets/images/ceshi.png'),
        isCrop: true,
        edg: 0,
        fullScreen: true,
        domId: '',
        imgType: 'image/png',
        imgBack: require('@/assets/images/22.png'),
        isRepeat: '',
        noRotation: false,
        backIsCenter: false,
        verticalDeductWidth: 10,
        verticalDeductHeight: 24,
        acrossDeductWidth: 30,
        acrossDeductHeight: 20,
        recoverPoints: []
      }
    }
  },
  methods: {
    onClick () {
      this.$refs.vueSignatureRef.confirm()
        .then(res => {
          this.imagesSRC = res.base64
        })
    },
    recover () {
      let list = [] // 不建议为空
      this.$refs.vueSignatureRef.recoverDraw(list)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
html,body{
 height: 100vh;
 padding: 0;
 margin: 0;
}
</style>
<style scoped>
div{
 height: 100vh;
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
