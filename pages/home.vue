<template>
  <div>
    <h1>Login Succeeded!</h1>
    <button>Logout</button>
  </div>
</template>
<script>
export default {
  methods: {
    logout() {
      this.$auth.logout();
    },
  },
};
</script>