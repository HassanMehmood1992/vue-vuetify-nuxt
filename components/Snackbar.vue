<template>
  <v-snackbar v-model="snackbar.show" top right :timeout="snackbar.timeout">
    {{ snackbar.text }}
    <v-btn dark text class="caption" @click="snackbar.show = false">close</v-btn>
  </v-snackbar>
</template>

<script>
export default {
  props: ['snackbar'],
  data: () => ({})
}
</script>