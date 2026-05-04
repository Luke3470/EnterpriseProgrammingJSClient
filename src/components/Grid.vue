<template>
  <v-row>
    <v-col v-for="book in books" :key="book.id" cols="12" sm="6" md="4">
      <Card
          :book="book"
          :flipped="flippedCards.has(book.id)"
          :format="format"
          :raw="rawBooks[book.id]"
          :format-book="formatBook"
          :truncate="truncate"
          @flip="$emit('flip', book.id)"
          @edit="$emit('edit', book)"
          @delete="$emit('delete', book)"
      />
    </v-col>
  </v-row>
</template>

<script>
import Card from "./Card.vue"

export default {
  components: { Card },
  props: ["books", "flippedCards", "format", "rawBooks", "formatBook", "truncate"]
}
</script>