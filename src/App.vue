<template>
  <v-app>
    <v-container class="py-6">

      <Header
          :format="format"
          :formats="formats"
          @add-book="openInsert"
          @format-change="val => { format = val; resetAndLoad() }"
      />

      <Filters
          :filters="filters"
          :loading="loading"
          @search="resetAndLoad"
          @clear="clearFilters"
          @update-filters="val => filters = val"
      />

      <v-progress-linear v-if="loading" indeterminate class="mb-4" />

      <Grid
          :books="books"
          :flipped-cards="flippedCards"
          :format="format"
          :raw-books="rawBooks"
          @flip="flip"
          @edit="openEdit"
          @delete="confirmDelete"
          :format-book="formatBook"
          :truncate="truncate"
      />

      <v-row v-if="!loading && books.length === 0">
        <v-col class="text-center text-medium-emphasis py-12">
          No books found. Try adjusting your filters.
        </v-col>
      </v-row>

      <v-row class="mt-6" justify="center" v-if="totalPages > 1 || currentPage > 1">
        <v-col cols="auto">
          <v-pagination
              v-model="currentPage"
              :length="totalPages"
              :total-visible="7"
              @update:modelValue="loadBooks"
          />
        </v-col>
      </v-row>

      <EditBook
          :dialog="dialog"
          @save="saveDialog"
          @close="dialog.open = false"
      />

      <Delete
          :delete-dialog="deleteDialog"
          @confirm="executeDelete"
          @cancel="deleteDialog.open = false"
      />

      <Snackbar :snackbar="snackbar" />

    </v-container>
  </v-app>
</template>

<script>
import axios from "axios"
import { XMLParser } from "fast-xml-parser"
import YAML from "js-yaml"


import Header from "./components/Header.vue";
import Filters from "./components/Filters.vue";
import Grid from "./components/Grid.vue";
import EditBook from "./components/EditBook.vue";
import Delete from "./components/Delete.vue";
import Snackbar from "./components/Snackbar.vue";

const BASE_URL = "http://localhost:8080/EnterpriseProgrammingREST_war/bookAPI"

const EMPTY_FORM = () => ({
  id: "", title: "", author: "", date: "",
  genres: "", characters: "", synopsis: "", coverUrl: ""
})

export default {
  components: {
    Header,
    Filters,
    Grid,
    EditBook,
    Delete,
    Snackbar
  },

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
      rawBooks: {},

      filters: {
        id: "", books: "", genres: "", dateFrom: "", dateTo: ""
      },

      currentPage: 1,
      totalPages: 1,

      dialog: {
        open: false,
        mode: "insert",
        form: EMPTY_FORM(),
        saving: false,
        error: ""
      },

      deleteDialog: {
        open: false,
        book: null,
        deleting: false
      },

      snackbar: {
        show: false,
        message: "",
        color: "success"
      }
    }
  },

  mounted() {
    this.loadBooks()
  },

  methods: {
    buildParams() {
      const p = { page: this.currentPage }
      if (this.filters.id) p.id = this.filters.id
      if (this.filters.books) p.books = this.filters.books
      if (this.filters.genres) p.genres = this.filters.genres
      if (this.filters.dateFrom) p.dateFrom = this.filters.dateFrom
      if (this.filters.dateTo) p.dateTo = this.filters.dateTo
      return p
    },

    resetAndLoad() {
      this.currentPage = 1
      this.loadBooks()
    },

    clearFilters() {
      this.filters = { id: "", books: "", genres: "", dateFrom: "", dateTo: "" }
      this.resetAndLoad()
    },

    async loadBooks() {
      this.loading = true
      this.flippedCards = new Set()

      try {
        const res = await axios.get(BASE_URL, {
          headers: { Accept: this.format },
          params: this.buildParams()
        })

        const data = res.data

        if (this.format === "application/xml") {
          const parser = new XMLParser({ ignoreAttributes: false })
          this.rawBooks = {}
          const parsed = parser.parse(data)
          let books = parsed.response?.books ?? []
          books = Array.isArray(books) ? books : [books]

          books.forEach(b => {
            this.rawBooks[String(b.id)] = this.getRawSegmentFromXML(data, b.id)
          })

          this.books = books
          this.totalPages = parsed.response?.totalPages ?? 1

        } else if (this.format === "application/json") {
          this.rawBooks = {}
          const books = data.books || []
          books.forEach(b => {
            this.rawBooks[String(b.id)] = JSON.stringify(b, null, 2)
          })
          this.books = books
          this.totalPages = data.totalPages ?? 1

        } else {
          const cleanYaml = data.replace(/^!!.*\n/, "")
          const parsed = YAML.load(cleanYaml)
          const books = parsed?.books || []
          this.books = books
          this.totalPages = parsed?.totalPages ?? 1

          this.rawBooks = {}
          books.forEach(b => {
            this.rawBooks[String(b.id)] = YAML.dump(b)
          })
        }

      } finally {
        this.loading = false
      }
    },

    flip(id) {
      this.flippedCards.has(id)
          ? this.flippedCards.delete(id)
          : this.flippedCards.add(id)

      this.flippedCards = new Set(this.flippedCards)
    },

    formatBook(book) {
      if (this.format === "application/json")
        return JSON.stringify(book, null, 2)

      if (this.format === "application/x-yaml")
        return YAML.dump(book)

      return this.rawBooks[String(book.id)] || ""
    },

    getRawSegmentFromXML(fullXml, id) {
      const segments = fullXml.match(/<books>[\s\S]*?<\/books>/g) || []
      const target = String(id)
      return segments.find(seg => {
        const match = seg.match(/<id>(.*?)<\/id>/)
        return match && String(match[1]).trim() === target })
          || "Book XML not found"
    },

    truncate(text, len) {
      return text && text.length > len
          ? text.substring(0, len) + "..."
          : text
    },

    openInsert() {
      this.dialog = {
        ...this.dialog,
        mode: "insert",
        form: EMPTY_FORM(),
        open: true,
        error: ""
      }
    },

    openEdit(book) {
      this.dialog = {
        ...this.dialog,
        mode: "edit",
        form: { ...book },
        open: true,
        error: ""
      }
    },

    async saveDialog() {
      this.dialog.saving = true
      try {

        let payload
        let body
        let headers = {}

        if (this.dialog.mode === "insert") {
          const { id, ...withoutId } = basePayload
          payload = withoutId
        }

        else {
          payload = basePayload
        }

        if (this.format === "application/json") {
          body = JSON.stringify(payload)
          headers["Content-Type"] = "application/json"
          headers["Accept"] = "application/json"
        }

        else if (this.format === "application/xml") {
          const builder = new XMLBuilder({
            format: true,
            ignoreAttributes: false
          })

          body = builder.build({
            book: payload
          })

          headers["Content-Type"] = "application/xml"
          headers["Accept"] = "application/xml"
        }

        else if (this.format === "application/x-yaml") {
          body = YAML.dump(payload, {
            lineWidth: -1,
            noRefs: true
          })

          headers["Content-Type"] = "application/x-yaml"
          headers["Accept"] = "application/x-yaml"
        }

        if (this.dialog.mode === "edit") {
          await axios.put(`${BASE_URL}`, payload)
        } else {
          await axios.post(BASE_URL, payload)
        }

        this.dialog.open = false
        this.loadBooks()

      } catch (err) {
        this.dialog.error = err.message
      }

      this.dialog.saving = false
    },

    confirmDelete(book) {
      this.deleteDialog = { open: true, book, deleting: false }
    },

    async executeDelete() {
      this.deleteDialog.deleting = true

      try {
        await axios.delete(`${BASE_URL}?id=${this.deleteDialog.book.id}`)
        this.notify(
            `"${this.deleteDialog.book.title}" deleted`,
            "error"
        )
        this.deleteDialog.open = false
        this.loadBooks()
      } catch (err) {
        this.notify(err.response?.data?.message ?? "Delete failed", "error")
      } finally {
        this.deleteDialog.deleting = false
      }
    },

    notify(message, color = "success") {
      this.snackbar.message = message
      this.snackbar.color = color
      this.snackbar.show = true
    }
  }
}
</script>