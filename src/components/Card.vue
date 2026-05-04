<template>
  <div class="flip-card">
    <div class="flip-inner" :class="{ flipped }">

      <div class="flip-front">
        <v-card elevation="3">
          <v-img :src="book.coverUrl" height="220" cover />
          <v-card-title>{{ book.title }}</v-card-title>
          <v-card-subtitle>{{ book.author }}</v-card-subtitle>

          <v-card-text>
            <div><strong>Published:</strong> {{ book.date }}</div>
            <div>{{ truncate(book.synopsis, 120) }}</div>
          </v-card-text>

          <v-card-actions>
            <v-btn @click="$emit('flip')">View Raw</v-btn>
            <v-spacer />
            <v-btn color="warning" variant="tonal" icon="mdi-pencil" @click="$emit('edit', book)" />
            <v-btn color="error" variant="tonal" icon="mdi-delete" @click="$emit('delete', book)" />
          </v-card-actions>
        </v-card>
      </div>

      <div class="flip-back">
        <v-card>
          <v-card-title>Raw Data</v-card-title>
          <v-card-text>
            <pre class="pre-style" >{{ formatBook(book) }}</pre>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="$emit('flip')">Back</v-btn>
          </v-card-actions>
        </v-card>
      </div>

    </div>
  </div>
</template>
<style scoped>
.flip-card {
  perspective: 1000px;
  height: 500px;
}

.flip-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.8s;
  transform-style: preserve-3d;
}

.flip-inner.flipped {
  transform: rotateY(180deg);
}

.flip-front,
.flip-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  top: 0;
  left: 0;
}

.flip-back {
  transform: rotateY(180deg);
}

.pre-style {
  padding: 12px;
  max-height: 360px;
  overflow: auto;
  font-size: 11px;
  white-space: pre-wrap;
  word-break: break-all;
  margin: 0;
}
</style>
<script>
export default {
  props: ["book", "flipped", "format", "formatBook", "truncate"]
}
</script>