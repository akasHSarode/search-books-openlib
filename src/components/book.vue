<template>
  <div class="book" @click="openBookDetails(book)">
    <img 
      :src="`https://covers.openlibrary.org/b/isbn/${book.isbn}-S.jpg`" 
      class="cover" alt="no-img" 
    />

    <section class="info">
      <h3 class="title">{{ book.title }}</h3>
      <p class="author">By {{ book.author }}</p>
    </section>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
  @Prop() book!: any;

  openBookDetails(book: any) {
    console.log('open book ', book)
    this.$emit('open-book-details', book);
  }
}
</script>

<style scoped lang="scss">
  .book {
    padding: 10px 15px;
    border: 1px solid lightgray;
    border-radius: 4px;
    margin-top: 20px;
    margin-bottom: 20px;

    display: flex;
    gap: 10px;

    p {
      text-align: left
    };

    &:hover, &.selected {
      transform: scale(1.05);
      transition: all .2s ease-in-out;
      box-shadow: 5px 5px lightgray;
      cursor: pointer;
    }

    .cover {
      height: 70px;
      width: 50px;
    }

    .info {
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .title {
      font-size: 18px;
      font-weight: 500;
    }

    .author {
      font-size: 14px;
      color: gray
    }
  }
</style>
