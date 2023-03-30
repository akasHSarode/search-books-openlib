<template>
  <div class="home max-width">
    <div class="main">
      <div class="search-box">
        <span class="brand is-size-4">
          Book Library
        </span>

        <b-input size="is-medium" placeholder="Search books" v-model="query"/>
        <b-button size="is-medium" @click="search" label="Search"/>
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
          @open-book-details="openBookDetails"
        />
      </section>
      <p v-else class="no-books">
        There are no books, Try searching for something...
      </p>

      <b-modal v-model="modalOpen" class="selected-book">
        <div class="card">
          <b-image 
            v-if="!selectedBook.imageUrl"
            :src="require('@/assets/Default_image.jpeg')" 
            alt="no image" 
            class="cover"
          />
          <b-image 
            v-else
            :src="selectedBook.imageUrl" 
            class="cover" 
            alt="no-img"
          />

          <div class="content">
            <div class="title">
              <p class="title is-4">
                {{ selectedBook.title }}
                <span v-if="selectedBook.publishDate">
                  - ({{ selectedBook.publishDate }})
                </span>
              </p>
              <p class="subtitle is-6">{{ selectedBook.author }}</p>
            </div>

            <div class="description">
              <p v-if="selectedBook.description" class="subtitle is-6">{{ selectedBook.description }}</p>
              <p v-else class="subtitle is-6">No Description Available</p>
            </div>
                    
            <b-taglist v-if="selectedBook.subjects">
              <b-tag v-for="(subject, index) in selectedBook.subjects.slice(0,9)" :key="index">
                {{ subject }}
              </b-tag>
            </b-taglist>
          </div>
        </div>
      </b-modal>
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
  openApiSearchUrl = 'https://openlibrary.org/search.json';

  query = '';
  books: any[] = [];
  loading = false;

  selectedBook: any = {};

  modalOpen = false;

  openBookDetails(book: any) {
    this.selectedBook = book;
    this.getDetailsOfBook();
    this.coverUrl();
    this.modalOpen = true;
  }

  getDetailsOfBook() {
    fetch(`https://openlibrary.org${this.selectedBook.key}.json`)
      .then((res: any) => res.json())
      .then(({ description, first_publish_date, subjects }: any) => {
        // some books have description in following format
        if(typeof description !== 'string') {
          description = description.value
        }

        this.$set(this.selectedBook, 'description', description ?? '');
        this.$set(this.selectedBook, 'publishDate', first_publish_date ?? '');
        this.$set(this.selectedBook, 'subjects', subjects ?? []);
      })
      .catch((e: any) => console.log('error, ', e));
  }

  async coverUrl() {
    const url = `https://covers.openlibrary.org/b/isbn/${this.selectedBook.cover}-L.jpg`;

    try {
      const response = await fetch(url);
      const blob = await response.blob();
      if (blob.type === '') {
        throw new Error('Invalid image');
      }

      const imageUrl = URL.createObjectURL(blob);
      this.$set(this.selectedBook, 'imageUrl', imageUrl);
    } catch(error) {
      // log error
    }    
  }

  async search() {
    this.loading = true;
    this.books = [];

    let query = this.query;

    // limit records to 10
    query += '&limit=10';

    const url = query ? `${this.openApiSearchUrl}?q=${query}` : this.openApiSearchUrl;

    try {
      const response = await fetch(url);
      const { docs } = await response.json();
      const books = docs.map((book: any) => {
        const [author] = book.author_name;
        const [cover] = book?.isbn ?? [];

        return {
          ...book,
          author,
          cover
        };
      });
      this.books = books;
    } catch (error) {
      console.error('Error in search api', error);
    } finally {
      this.loading = false;
    }
  }
}
</script>

<style lang="scss" scoped>
.home {
  display: flex;
}

.search-box {
  width: 1000px;
  display: flex;
  align-items: center;

  margin-top: 20px;
  padding: 40px 15px;
  background: #eee;

  position: sticky;
  top: 0;
  z-index: 10;

  span.brand {
    margin-right: 20px;
    font-weight: bold;
  }

  // search input box
  .control {
    // take all available space
    flex: 1;
  }

  button {
    margin-left: 10px;
  }
}

p.no-books {
  margin-top: 140px;
}

::v-deep {
  .modal.selected-book {
    .card {
      padding: 25px;
      display: flex;
      gap: 25px;

      .content {
        display: flex;
        flex-direction: column;
        flex: 2;
        text-align: left;

        .description {
          height: 300px;
          overflow: hidden;
          text-overflow: ellipsis;
          -webkit-line-clamp: 4;
          display: -webkit-box;
          -webkit-box-orient: vertical;
          white-space: break-spaces;
        }

        .tags {
          margin-top: 1rem;
        }
      }
    }

    .b-image-wrapper.cover {
      width: 330px;
      height: 480px;
      overflow: hidden;
      margin: 0;

      img {
        min-height: 480px;
      }
    }
  }
}
</style>
