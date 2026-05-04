<template>
  <v-app>
    <v-container class="py-6">

      <v-row class="mb-4" align="center">
        <v-col cols="12" md="6">
          <h1>Book Library</h1>
          <p class="text-medium-emphasis">
            Browse books from REST API
          </p>
        </v-col>

        <v-col cols="12" md="6" class="text-right">
          <v-select
              v-model="format"
              :items="formats"
              label="Response Format"
              density="comfortable"
              @update:modelValue="loadBooks"
          />
        </v-col>
      </v-row>

      <v-progress-linear v-if="loading" indeterminate class="mb-4" />

      <v-row>
        <v-col
            v-for="book in books"
            :key="book.id"
            cols="12"
            sm="6"
            md="4"
        >
          <div class="flip-card">
            <div class="flip-inner" :class="{ flipped: flippedCards.has(book.id) }">

              <div class="flip-front">
                <v-card class="h-100" elevation="3">
                  <v-img :src="book.coverUrl" height="260" cover />
                  <v-card-title>{{ book.title }}</v-card-title>
                  <v-card-subtitle>{{ book.author }} </v-card-subtitle>
                  <v-card-text>
                    <div><strong>Published:</strong> {{ book.date }}</div>
                    <div class="mt-2">
                      {{ truncate(book.synopsis, 120) }}
                    </div>
                    <v-btn
                        class="mt-3"
                        size="small"
                        color="primary"
                        @click="flip(book.id)"
                    >
                      View Raw
                    </v-btn>
                  </v-card-text>
                </v-card>
              </div>
              <div class="flip-back">
                <v-card class="h-100" elevation="6">
                  <v-card-title>Raw Data</v-card-title>
                  <v-card-text  class="pa-0">
                    <pre class="pre-style language-xml" v-if="format==='application/xml'"><code class="language-xml">{{ formatBook(book) }}</code></pre>
                    <pre class="pre-style language-json" v-if="format==='application/json'"><code class="language-json">{{ formatBook(book) }}</code></pre>
                    <pre class="pre-style language-yaml" v-if="format==='application/x-yaml'"><code class="language-yaml">{{ formatBook(book) }}</code></pre>
                  </v-card-text>
                  <v-card-actions>
                    <v-spacer />
                    <v-btn color="red" variant="text" @click="flip(book.id)">
                      Back
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </div>
            </div>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
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

.pre-style{
  padding:12px;
  max-height:380px;
  overflow:auto;
  font-size:11px;
  white-space:pre-wrap;
  word-break:break-all;
  margin:0;
}

.flip-back {
  transform: rotateY(180deg);
}
</style>
<script>
import axios from "axios"
import { XMLParser } from "fast-xml-parser"
import YAML from "js-yaml"
import Prism from "prismjs"
import "prismjs/themes/prism.css"
import "prismjs/components/prism-json"
import "prismjs/components/prism-yaml"
import "prismjs/components/prism-xml-doc"

const BASE_URL = "http://localhost:8080/EnterpriseProgrammingREST_war/bookAPI"

export default {
  data() {
    return {
      books: [],
      loading: false,
      format: "application/xml",
      formats: [
        { title: "XML", value: "application/xml" },
        { title: "JSON", value: "application/json" },
        { title: "YAML", value: "application/x-yaml" }
      ],
      flippedCards: new Set(),
      rawBooks: {}
    }
  },
  watch: {
    flippedId() {
      this.$nextTick(() => {
        Prism.highlightAll();
      });
    }
  },

  mounted() {
    this.loadBooks()
  },

  methods: {
    async loadBooks() {
      this.loading = true

      try {
        const res = await axios.get(`${BASE_URL}`, {
          headers: {
            Accept: this.format
          }
        })

        let data = res.data

        if (this.format === "application/xml") {
          const parser = new XMLParser({ignoreAttributes: false})

          this.rawBooks = {}

          const parsed = parser.parse(data)
          let books = parsed.response.books
          books = Array.isArray(books) ? books : [books]

          books.forEach((b) => {
            this.rawBooks[String(b.id)] = this.getRawSegmentFromXML(data, b.id)
          })

          this.books = books
        }

        else if (this.format === "application/json") {
          this.rawBooks = {}

          const books = data.books || []
          books.forEach(b => {
            this.rawBooks[b.id] = JSON.stringify(b, null, 2)
          })

          this.books = books
        }


        else if (this.format === "application/x-yaml") {
          const cleanYaml = data.replace(/^!!.*\n/, "");

          const parsed = YAML.load(cleanYaml);

          const books = parsed?.books || [];
          this.books = Array.isArray(books) ? books : [books];

          this.rawBooks = {};
          this.books.forEach(b => {
            this.rawBooks[String(b.id)] = YAML.dump(b, {
              lineWidth: -1,
              noRefs: true,
              flowLevel: -1,
              quotingType: '"',
              forceQuotes: false
            })
          });
        }

      } catch (err) {
        console.error("Failed to load books:", err)
        this.books = []
      }

      this.loading = false
    },

    flip(id) {
      if (this.flippedCards.has(id)) {
        this.flippedCards.delete(id)
      } else {
        this.flippedCards.add(id)
      }
      this.flippedCards = new Set(this.flippedCards)
    },

    formatBook(book) {
      if (this.format === "application/json") {
        return JSON.stringify(book, null, 2)
      }

      if (this.format === "application/x-yaml") {
        return YAML.dump(book)
      }

      if (this.format === "application/xml") {
        return this.formatXML(this.rawBooks[String(book.id)])
      }

      return ""
    },

    truncate(text, len) {
      if (!text) return ""
      return text.length > len ? text.substring(0, len) + "..." : text
    },

    getRawSegmentFromXML(fullXml, id) {
      const segments = fullXml.match(/<books>[\s\S]*?<\/books>/g) || []
      const target = String(id)
      return segments.find(seg => {
        const match = seg.match(/<id>(.*?)<\/id>/)
        return match && String(match[1]).trim() === target
      }) || "Book XML not found"
    },

    formatXML(xml) {
      try {
        const PADDING = "  ";
        const reg = /(>)(<)(\/*)/g;
        let formatted = "";
        let pad = 0;

        xml = xml.replace(reg, "$1\n$2$3");

        xml.split("\n").forEach((node) => {
          let indent = 0;

          if (node.match(/^<\/\w/)) pad -= 1;
          if (pad > 0) indent = pad;

          formatted += PADDING.repeat(indent) + node + "\n";

          if (node.match(/^<\w[^>]*[^\/]>$/)) pad += 1;
        });

        return formatted.trim();
      } catch (e) {
        return xml;
      }
    }
  }
}
</script>