<template>
  <div>
    <h3>
      welcome, <strong>{{ username }}</strong>
    </h3>
  </div>
</template>

<script>
export default {
  data() {
    return {
      username: ''
    }
  },
  created() {
    this.username = window.sessionStorage.getItem('username')
  }
}
</script>

<style lang="less" scoped></style>
