<template>
  <div class="home max-width">
    <div class="main">
      <!-- search box -->
      <div class="search-box flex">
          <b-input size="is-large" placeholder="Search books" v-model="query" />
          <b-button size="is-large" @click="search" label="Search"/>
      </div>
      
      <!-- list of books -->
      <b-loading 
        :is-full-page="false" 
        v-model="loading" 
        v-if="loading"
      />
      <section v-else-if="!loading && books.length > 0" class="books-list">
        <Book 
          v-for="book in books" 
          :key="book.key" 
          :book="book" 
          :class="{ selected: book.key === selectedBook.key }"
            @open-book-details="openSidebar"
        />
      </section>
      <p v-else class="no-books">
        There are no books, Try searching for something...
      </p>
    </div>

    <div class="sidebar">
      <b-sidebar
        type="is-light"
        :fullheight="true"
        :fullwidth="false"
        :overlay="false"
        :right="true"
        v-model="sidebarOpen"
        :can-cancel="false"
      >
      <img 
        :src="`https://covers.openlibrary.org/b/isbn/${selectedBook.isbn}-L.jpg`" 
        class="cover" alt="no-img" 
      />

      <p class="title">{{ selectedBook.title }}</p>
      <p class="author"><b>By</b> {{ selectedBook.author }}</p>
    </b-sidebar>

    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Book from '@/components/book.vue'

@Component({
  components: {
    Book
  },
})
export default class HomeView extends Vue {
  // sidebarOpen = false;
  openApiSearchUrl = 'https://openlibrary.org/search.json';

  query = '';
  books: any[] = [];
  loading = false;
  selectedBook: any = {};

  get sidebarOpen() {
    return (Object.keys(this.selectedBook).length > 0)
  }

  async search() {
    this.loading = true;
    this.books = [];

    let query = this.query;

    // limit records to 10
    query += '&limit=10';

    const url = `${this.openApiSearchUrl}?q=${query}`;
    this.books = await fetch(url)
      .then(res => res.json())
      .then(({ docs }) => docs.map((book: any) => {
        const [author] = book.author_name;
        const [isbn] = book?.isbn ?? [];

        return ({
          ...book,

          author,
          isbn
        });
      }))
      .catch(e => {
        console.log('error in search api', e);
      })
      .finally(() => {
        this.loading = false;
      })
  }

  openSidebar(book: any) {
    this.selectedBook = book;
  }
}
</script>

<style lang="scss" scoped>
.home {
  display: flex;
}

.search-box {
  margin-top: 24px;
}

p.no-books {
  margin-top: 140px;
}

::v-deep {
  .search-box {
    position: relative;
    width: 1000px;
    div.control {
      width: 100%;
    }

    button {
      margin-left: 10px;
    }
  }
}

::v-deep {
  .b-sidebar .sidebar-content {
    width: 330px;
    padding: 15px;

    .title {
      color: yellow
    }
  }
}

.b-sidebar .sidebar-content {
    width: 330px;
    padding: 15px;
  }

// .b-sidebar, .sidebar-content {
//     width: 500px;
//   }
</style>
