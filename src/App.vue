<template>
  <div id="app">
    <div class="middle" style="height: 800px;overflow: auto;">
      <div class="inner" style="height: 900px;padding-top: 100px;padding-left: 200px;">
        <popover trigger="click" :value="true" >
          <button>按钮</button>
          <div slot='content' >测试一哈子</div>
        </popover>
      </div>
    </div>
  </div>
</template>

<script>
import popover from './components/popover'
export default {
  name: 'App',
  components: {
    popover
  },
  data () {
    return {
      placement: 'bottom-end'
    }
  }
}
</script>

<style>
#app {
  text-align: center;
  height: 700px;
  overflow: auto;
}
body {
  padding: 0;
  margin: 0;
}
</style>
