<template>
  <div>
    <div class="book" @click="openBookDetails(book)">
      <b-image 
        v-if="!imageUrl"
        :src="require('@/assets/Default_image.jpeg')" 
        alt="no image" 
        class="cover"
      />
      <b-image 
        v-else
        
        :src="imageUrl" 
        class="cover" 
        alt="no-img"
      />

      <section class="info">
        <h3 class="title">{{ book.title }}</h3>
        <p class="author">By {{ book.author }}</p>
      </section>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
  @Prop() book!: any;

  imageUrl = '';

  openBookDetails(book: any) {
    this.$emit('open-book-details', book);
  }

  async coverUrl() {
    const url = `https://covers.openlibrary.org/b/isbn/${this.book.cover}-S.jpg`;

    try {
      const response = await fetch(url);
      const blob = await response.blob();
      if (blob.type === '') {
        throw new Error('Invalid image');
      }

      const imageUrl = URL.createObjectURL(blob);
      this.imageUrl = imageUrl;
    } catch(error) {
      // log error
    }    
  }

  created() {
    this.coverUrl();
  }
}
</script>

<style scoped lang="scss">
.book {
  margin: 20px 0;
  padding: 10px 15px;
  border: 1px solid lightgray;
  border-radius: 4px;
  width: 100%;
  display: flex;
  gap: 10px;

  &:hover {
    transform: scale(1.05);
    transition: all .2s ease-in-out;
    box-shadow: 5px 5px lightgray;
    cursor: pointer;
  }

  p {
    text-align: left
  };

  .cover {
    height: 70px;
    width: 50px;
    overflow: hidden;
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
