<template>
  <section
    v-if="product"
    v-editable="story"
    class="text-gray-700 body-font overflow-hidden bg-white"
  >
    <div
      class="container px-5 py-24 mx-auto text-center flex flex-col items-center storyblok__outline"
    >
      <h1 class="text-4xl font-bold mb-8">
        {{ story.content ? story.content.name : product.title }}
      </h1>
      <div>
        <span class="text-3xl text-primary text-left leading-tight h-3 block">
          <div class="leading-relaxed" v-html="richtext" />
        </span>
      </div>
    </div>
    <div class="container px-5 py-24 mx-auto">
      <div class="lg:w-4/5 mx-auto flex flex-wrap">
        <img
          v-if="getFirstImage"
          :alt="getFirstImage.altText"
          class="lg:w-1/2 w-full object-cover object-center rounded border border-gray-200"
          :src="getFirstImage.transformedSrc"
        />
        <div class="lg:w-1/2 w-full lg:pl-10 lg:py-6 mt-6 lg:mt-0">
          <p class="text-sm title-font text-gray-500 tracking-widest">
            <span
              v-for="category in product.collections.edges"
              :key="category.node.title"
              >{{ category.node.title }} -</span
            >
          </p>
          <h1 class="text-gray-900 text-3xl title-font font-medium mb-1">
            {{ product.title }}
          </h1>
          <p class="leading-relaxed" v-html="product.description"></p>
          <span class="block mt-4 pb-5 border-b-2 border-gray-200 mb-5"></span>
          <div class="flex">
            <span class="title-font font-medium text-2xl text-gray-900">
              {{
                `${product.priceRange.maxVariantPrice.amount} ${product.priceRange.maxVariantPrice.currencyCode}`
              }}
            </span>
            <!-- this is handle by the checkout graphql api -->
            <a
              href="#"
              class="flex ml-auto text-white bg-primary border-0 py-2 px-6 focus:outline-none hover:bg-gray-800 rounded"
              >Add to Cart</a
            >

            <a
              :href="product.addToWishlistUrl"
              class="rounded-full w-10 h-10 bg-gray-200 p-0 border-0 inline-flex items-center justify-center text-gray-500 ml-4"
            >
              <svg
                fill="currentColor"
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                class="w-5 h-5"
                viewBox="0 0 24 24"
              >
                <path
                  d="M20.84 4.61a5.5 5.5 0 00-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 00-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 000-7.78z"
                />
              </svg>
            </a>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { getProductBySlug } from '~/plugins/graphql-shopify'

export default {
  async asyncData(context) {
    try {
      const { data } = await context.app.$storyapi.get(
        `cdn/stories/product/${context.params.slug}`,
        {
          version: 'draft',
        }
      )

      const res = await getProductBySlug(context.params.slug)

      console.log(res)
      return {
        product: res.productByHandle,
        story: data.story,
      }
    } catch (e) {
      console.log(e)
    }
  },
  data: () => ({
    product: null,
    story: {},
  }),
  computed: {
    richtext() {
      return this.$storyapi.richTextResolver.render(
        this.story.content.description
      )
    },
    getFirstImage() {
      return this.product ? this.product.images.edges[0].node : null
    },
  },
  mounted() {
    this.$storybridge.on(['input', 'published', 'change'], (event) => {
      if (event.action === 'input') {
        if (event.story.id === this.story.id) {
          this.story.content = event.story.content
        }
      } else if (!event.slugChanged) {
        window.location.reload()
      }
    })
  },
}
</script>
