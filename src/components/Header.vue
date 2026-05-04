<template>
  <v-row class="mb-4" align="center">
    <v-col cols="12" md="6">
      <h1>Book Library</h1>
    </v-col>

    <v-col cols="12" md="6" class="text-right d-flex align-center justify-end gap-3">
      <v-btn color="success" prepend-icon="mdi-plus" class="mr-3" @click="$emit('add-book')">
        Add Book
      </v-btn>

      <v-select
          :model-value="format"
          :items="formats"
          label="Response Format"
          density="comfortable"
          hide-details
          style="max-width:200px"
          @update:modelValue="$emit('format-change', $event)"
      />
    </v-col>
  </v-row>
</template>

<script>
export default {
  props: ["format", "formats"]
}
</script>