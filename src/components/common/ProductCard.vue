<template>
  <v-container fluid style="background-color: #EBF6FF;border-radius: 12px;" class="pa-4">
    <v-row>
      <v-col md="12">
        <v-img
          :src="item.image"
          :lazy-src="item.image"
          style="border-radius: 10px"
          height="200px"
        ></v-img>
        <p
          style="font-size: 135%; font-weight: 600; color: #242424"
          class="mt-4 mb-3"
        >
          {{ item.name }}
        </p>
        <p class="mb-1" style="font-size: 80%;color: #242424;"><b>Description</b></p>
        <p class="mb-1" style="font-size: 90%">{{ item.description }}</p>
        <v-btn
          :href="item.url"
          target="_blank"
          color="primary"
          depressed
          rounded
          class="mt-1"
          >Get Product Link</v-btn
        >

        <p class="mt-5"><b>Similer Products to {{ item.name }}</b></p>
        <v-container fluid class="pa-0 ma-0">
          <v-row>
            <v-col
              md="6"
              cols="6"
              v-for="(itemp, indexp) in item.similar_products.length>4?item.similar_products.slice(0,4):item.similar_products"
              :key="indexp"
            >
              <v-card
                style="border-radius: 12px;"
                elevation="0"
                :href="itemp.url"
                class="pa-2 text-center"
                target="_blank"
              >
                <v-img
                  :src="itemp.img"
                  style="border-radius: 10px"
                  :lazy-src="item.image"
                  height="100px"
                ></v-img>
                <p style="font-weight: 500;font-size: 90%;" class="mt-3 mb-0">
                  {{ itemp.name }}
                </p>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  props: ["item"],
  name: "ProductCard",
};
</script>

<style>
</style>