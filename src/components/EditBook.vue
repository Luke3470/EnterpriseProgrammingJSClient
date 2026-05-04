<template>
  <v-dialog v-model="dialog.open" max-width="640">
    <v-card>

      <v-card-text>
        <v-form ref="form">
          <v-text-field
              v-model="dialog.form.title"
              label="Title"
              :rules="[rules.required, rules.max255]"
          />
          <v-text-field
              v-model="dialog.form.author"
              label="Author"
              :rules="[rules.required, rules.max255]"
          />
          <v-text-field
              v-model="dialog.form.date"
              label="Date"
              type="date"
              :rules="[rules.dateRequired, rules.dateFormat, rules.validDate]"
          />
          <v-text-field
              v-model="dialog.form.genres"
              label="Genres"
              :rules="[rules.max255]"
          />
          <v-text-field
              v-model="dialog.form.coverUrl"
              label="Cover URL"
          />
          <v-textarea
              v-model="dialog.form.synopsis"
              label="Synopsis"
          />
        </v-form>
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
  props: ["dialog", "rules"],
}
</script>