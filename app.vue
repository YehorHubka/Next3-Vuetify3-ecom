<template>
  <v-app id="inspire">
    <v-app-bar>
      <v-toolbar-title>Ecommerce Website</v-toolbar-title>
    </v-app-bar>

    <v-main class="bg-grey-lighten-2">
      <v-container fluid>
        <v-row>
          <v-col cols="12" md="3">
            <v-card class="mx-auto mb-4" color="grey-lighten-4">
              <v-card-text>
                <v-text-field
                  density="compact"
                  variant="solo"
                  label="Search..."
                  append-inner-icon="mdi-magnify"
                  single-line
                  hide-details
                  v-model="name"
                ></v-text-field>
              </v-card-text>
            </v-card>
            <v-expansion-panels v-model="panel">
              <v-expansion-panel>
                <v-expansion-panel-title>
                  <h2>Filters</h2>
                </v-expansion-panel-title>
                <v-expansion-panel-text class="pt-4">
                  <v-radio-group v-model="sortBy">
                    <template v-slot:label>
                      <h3>Sort by:</h3>
                    </template>
                    <v-radio value="title">
                      <template v-slot:label>
                        <div>By <strong>name</strong></div>
                      </template>
                    </v-radio>
                    <v-radio value="price">
                      <template v-slot:label>
                        <div>By <strong>price</strong></div>
                      </template>
                    </v-radio>
                  </v-radio-group>
                  <v-radio-group v-model="order">
                    <template v-slot:label>
                      <h3>Sorting order:</h3>
                    </template>
                    <v-radio value="asending">
                      <template v-slot:label>
                        <div>By <strong>asending</strong></div>
                      </template>
                    </v-radio>
                    <v-radio value="deasending">
                      <template v-slot:label>
                        <div>By <strong>deasending</strong></div>
                      </template>
                    </v-radio>
                  </v-radio-group>
                </v-expansion-panel-text>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
          <v-col cols="12" md="9">
            <v-row>
              <v-col
                sm="6"
                md="4"
                lg="3"
                v-for="item in productsPerPage"
                :key="item.id"
              >
                <v-card>
                  <v-img :src="item.images[0]" height="200px" cover></v-img>
                  <v-card-title> {{ item.title }} </v-card-title>
                  <v-card-subtitle> {{ item.description }} </v-card-subtitle>
                  <v-card-title> ${{ item.price }} </v-card-title>
                  <v-btn color="orange-lighten-2" variant="text">
                    Read more
                  </v-btn>
                </v-card>
              </v-col>
            </v-row>
            <v-pagination
              v-model="page"
              class="mt-5"
              :length="paginationLength"
            ></v-pagination>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
const panel = ref([0]);
const page = ref(1);
const { data: products } = await useFetch(
  "https://dummyjson.com/products?limit=0"
);
const sortBy = ref("");
const order = ref("asending");
const data = ref(products.value.products);
const name = ref("");
console.log();

const filteredProducts = computed(() => {
  if (name.value) {
    return [...data.value].filter((i) => {
      return name.value
        .toLocaleLowerCase()
        .split(" ")
        .every((v) => i.title.toLocaleLowerCase().includes(v));
    });
  } else {
    return data.value;
  }
});

const limitPerPage = ref(10);

const productsPerPage = computed(() => {
  return filteredProducts.value.slice(
    0 + (page.value - 1) * limitPerPage.value,
    limitPerPage.value + (page.value - 1) * limitPerPage.value
  );
});

const paginationLength = computed(() => {
  return Math.ceil(filteredProducts.value.length / limitPerPage.value);
});

const dynamicSort = (property) => {
  let sortOrder = 1;
  if (property[0] === "-") {
    sortOrder = -1;
    property = property.substr(1);
  }
  return (a, b) => {
    const result =
      a[property] < b[property] ? -1 : a[property] > b[property] ? 1 : 0;
    return result * sortOrder;
  };
};

const sortProducts = () => {
  if (order.value == "deasending") {
    data.value.sort(dynamicSort("-" + sortBy.value));
  } else {
    data.value.sort(dynamicSort(sortBy.value));
  }
};

watch(sortBy, () => {
  sortProducts();
});
watch(order, () => {
  sortProducts();
});
</script>