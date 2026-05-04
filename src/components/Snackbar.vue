<template>
  <v-snackbar
      v-model="snackbar.show"
      :color="snackbar.color"
      timeout="3000"
      location="bottom right"
  >
    {{ snackbar.message }}
  </v-snackbar>
</template>

<script>
export default {
  props: ["snackbar"]
}
</script>