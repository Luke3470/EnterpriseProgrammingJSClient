<template>
  <v-dialog v-model="deleteDialog.open" max-width="400">
    <v-card>

      <v-card-title>Delete Book</v-card-title>

      <v-card-text>
        Are you sure you want to delete
        <strong>{{ deleteDialog.book?.title }}</strong>?
      </v-card-text>

      <v-card-actions>
        <v-btn @click="$emit('cancel')">Cancel</v-btn>
        <v-btn color="error" @click="$emit('confirm')" :loading="deleteDialog.deleting">
          Delete
        </v-btn>
      </v-card-actions>

    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: ["deleteDialog"]
}
</script>