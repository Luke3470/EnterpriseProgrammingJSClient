<template>
  <v-dialog v-model="dialog.open" max-width="640">
    <v-card>

      <v-card-title>
        {{ dialog.mode === 'edit' ? 'Edit Book' : 'Add Book' }}
      </v-card-title>

      <v-card-text>
        <v-text-field v-model="dialog.form.id" label="ID" />
        <v-text-field v-model="dialog.form.title" label="Title" />
        <v-text-field v-model="dialog.form.author" label="Author" />
        <v-text-field v-model="dialog.form.date" label="Date" type="date" />
        <v-text-field v-model="dialog.form.genres" label="Genres" />
        <v-text-field v-model="dialog.form.coverUrl" label="Cover URL" />
        <v-textarea v-model="dialog.form.synopsis" label="Synopsis" />

        <v-alert v-if="dialog.error" type="error">
          {{ dialog.error }}
        </v-alert>
      </v-card-text>

      <v-card-actions>
        <v-btn @click="$emit('close')">Cancel</v-btn>
        <v-btn @click="$emit('save')" :loading="dialog.saving">
          Save
        </v-btn>
      </v-card-actions>

    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: ["dialog"]
}
</script>