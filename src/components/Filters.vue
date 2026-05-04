<template>
  <v-card class="mb-6 pa-4" elevation="2">
    <v-row density="comfortable">

      <v-col cols="12" sm="6" md="3">
        <v-text-field v-model="local.id" label="Book ID" clearable />
      </v-col>

      <v-col cols="12" sm="6" md="3">
        <v-text-field v-model="local.books" label="Title search" clearable />
      </v-col>

      <v-col cols="12" sm="6" md="3">
        <v-text-field v-model="local.genres" label="Genre" clearable />
      </v-col>

      <v-col cols="12" sm="6" md="3">
        <v-text-field v-model="local.dateFrom" label="Date from" type="date" />
      </v-col>

      <v-col cols="12" sm="6" md="3">
        <v-text-field v-model="local.dateTo" label="Date to" type="date" />
      </v-col>

      <v-col cols="12" sm="6" md="3" class="d-flex align-center gap-2">
        <v-btn color="primary" @click="$emit('search')" :loading="loading">Search</v-btn>
        <v-btn variant="text" class="ml-2" @click="reset">Clear</v-btn>
      </v-col>

    </v-row>
  </v-card>
</template>

<script>
export default {
  props: ["filters", "loading"],
  data() {
    return {
      local: { ...this.filters }
    }
  },
  watch: {
    local: {
      deep: true,
      handler(val) {
        this.$emit("update-filters", val)
      }
    }
  },
  methods: {
    reset() {
      this.local = { id: "", books: "", genres: "", dateFrom: "", dateTo: "" }
      this.$emit("clear")
    }
  }
}
</script>