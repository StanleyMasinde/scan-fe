<script setup lang="ts">
const route = useRoute();

useSeoMeta({
  title: "Product information",
});
</script>

<template>
  <div class="pb-16">
    <div>
      <p class="text-lg">Showing product with GTIN: {{ route.params.gtin }}</p>
    </div>

    <!-- Product Information -->
    <section>
      <img
        src="/product.jpg"
        class="w-full h-80 object-contain"
        alt="Nike Running shoes."
      />

      <div>
        <div class="mb-2">
          <h4 class="text-xl text-primary">Sticky's</h4>
          <h2 class="text-2xl">
            Sticky's Traditional Strawberry Jam Low Sugar 500 Gram
          </h2>
        </div>

        <div class="grid grid-cols-2 gap-2 mb-2">
          <div class="bg-gray-300 p-2 rounded-lg">
            <h3 class="text-sm text-primary font-light">Category</h3>
            <p>10000217 Jams/Marmalades (Shelf Stable)</p>
          </div>

          <div class="bg-gray-300 p-2 rounded-lg">
            <h3 class="text-sm text-primary font-light">Net content</h3>
            <p class="text-sm">500 Gram</p>
          </div>

          <div class="bg-gray-300 p-2 rounded-lg font-bold col-span-2">
            <h3 class="text-sm text-primary font-light">Country of sale</h3>
            <p class="text-sm">Kenya</p>
          </div>
        </div>

        <hr />

        <div class="mt-2">
          <h2>Company information</h2>

          <div>
            <h3 class="text-2xl text-primary">GS1 Demo Account</h3>
            <p class="text-sm">
              Avenue Loise 326 Blue Tower, Brussels, Belgium
            </p>
          </div>
        </div>
      </div>
    </section>

    <div>
      <div class="flex items-center justify-center w-full my-5">
        <NuxtLink
          class="bg-primary text-white py-2 w-[97%] fixed bottom-16 text-center left-1.5 right-1.5 px-3 rounded-lg text-lg"
          to="/"
          >Scan Again</NuxtLink
        >
      </div>
    </div>
  </div>
</template>
