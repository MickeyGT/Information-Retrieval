<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" :clipped="$vuetify.breakpoint.lgAndUp" fixed app>
      <v-list>
        <v-list-tile v-for="item in items" :key="item.text" @click="changeCategory(item.category)">
          <v-list-tile-action>
            <v-icon :color="item.category === category ? 'primary' : 'black'">{{ item.icon }}</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title
              :class="item.category === category ? 'blue--text' : 'black--text'"
            >{{ item.text }}</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      color="red darken-3"
      dark
      flat
      app
      fixed
    >
      <v-toolbar-title style="width: 300px" class="ml-0 pl-3">
        <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
        <span class="hidden-sm-and-down">GameStore</span>
      </v-toolbar-title>
      <v-combobox
        v-model="search.model"
        :items="autocompleteItems"
        :search-input.sync="search.search"
        label="Search"
        append-icon="search"
        class="mx-3"
        clearable
        flat
        hide-no-data
        hide-details
        solo-inverted
      ></v-combobox>
      <v-spacer></v-spacer>
      <v-btn class="white--text" light flat @click="onCart">
        <v-icon color="white">shopping_cart</v-icon>
        Cart ({{ cartSize }})
      </v-btn>
    </v-toolbar>
    <v-content>
      <v-container fluid>
        <v-layout row wrap>
          <v-flex xs12 sm6 md4 lg4 v-for="item in shopItems" :key="item.category">
            <v-card class="ma-4">
              <v-layout>
                <v-flex xs5>
                  <v-img :src="item.image" height="125px" contain></v-img>
                </v-flex>
                <v-flex xs7>
                  <v-card-title primary-title>
                    <div>
                      <div class="headline">{{item.name}}</div>
                      <div>{{item.description.length >= 50 ? item.description.substr(0, 50) + '...' : item.description}}</div>
                      <div>
                        <span class="mt-2 title text-xs-left red--text">${{item.price}}</span>
                      </div>
                    </div>
                  </v-card-title>
                  <v-card-title>
                    <v-text-field
                      v-model="item.currentQuantity"
                      class="spin-input"
                      readonly
                      type="number"
                      append-outer-icon="add"
                      @click:append-outer="item.currentQuantity++"
                      prepend-icon="remove"
                      @click:prepend="item.currentQuantity = item.currentQuantity - 1 ? item.currentQuantity - 1 : item.currentQuantity"
                    ></v-text-field>
                  </v-card-title>
                </v-flex>
              </v-layout>
              <v-divider light></v-divider>
              <v-card-actions>
                <v-btn flat color="warning" @click="showReviewDialog(item.video)">View review</v-btn>

                <v-spacer></v-spacer>

                <v-btn color="primary" @click="addItem(item)">Add to cart</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <v-dialog v-model="dialog" width="800px">
      <v-card>
        <v-card-title class="grey lighten-4 py-4 title">Cart</v-card-title>
        <v-container grid-list-sm class="pa-4">
          <v-layout row wrap>
            <v-flex xs12 align-center justify-space-between>
              <v-data-table
                :headers="cartDialog.headers"
                :items="cartDialog.items"
                class="elevation-1"
                hide-actions
              >
                <template v-slot:items="props">
                  <td>{{ props.item.name }}</td>
                  <td>{{ props.item.quantity }}</td>
                  <td>{{ props.item.price }}</td>
                </template>
                <template v-slot:no-data>
                  <p class="text-xs-center">The cart is empty</p>
                </template>
              </v-data-table>

              <p
                class="headline text-xs-right mt-3"
              >Total: ${{Math.floor(cartDialog.total)}}.{{Math.floor(cartDialog.total*100)%100}}</p>
            </v-flex>
          </v-layout>
        </v-container>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn flat color="primary" @click="clearCart">Checkout</v-btn>
          <v-btn flat @click="dialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="reviewDialog.show" width="800px" height="500px" scrollable="false">
      <v-card dark>
        <v-card-text>
          <iframe allowfullscreen width="765" height="500" :src="reviewDialog.url"></iframe>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import products from "./products";

export default {
  data: () => ({
    dialog: false,
    drawer: null,
    count: 0,
    search: {
      items: {
        laptops: [],
        keyboards: [],
        mice: [],
        phones: []
      },
      search: "",
      model: ""
    },
    items: [
      { category: "laptops", icon: "games", text: "Role Playing Games" },
      { category: "keyboards", icon: "directions_car", text: "Racing Games" },
      { category: "mice", icon: "people", text: "Multiplayer Games" },
      { category: "phones", icon: "center_focus_weak", text: "Shooter Games" }
    ],
    category: "laptops",
    products: products,
    cartSize: 0,
    cartDialog: {
      headers: [
        { text: "Product", value: "name", align: "left" },
        { text: "Quantity", value: "quantity", align: "left" },
        { text: "Price", value: "price", align: "left" }
      ],
      items: [],
      total: 0
    },
    reviewDialog: {
      show: false,
      url: ""
    }
  }),
  mounted() {
    for (const item of this.items) {
      for (const prod of this.products[item.category]) {
        this.search.items[item.category].push(prod.name);
      }
    }
    // eslint-disable-next-line
    console.log(this.search);
  },
  computed: {
    shopItems() {
      if (!this.search.model) {
        return this.products[this.category];
      }

      const found = this.products[this.category].find(
        item => item.name.toLowerCase() === this.search.model.toLowerCase()
      );
      if (found) {
        return [found];
      }

      return this.products[this.category].filter(
        item =>
          !this.search.model ||
          item.name.toLowerCase().includes(this.search.search.toLowerCase()) ||
          item.description
            .toLowerCase()
            .includes(this.search.search.toLowerCase())
      );
    },
    autocompleteItems() {
      if (!this.search.search) {
        return this.search.items[this.category];
      }

      return [this.search.search || "", ...this.search.items[this.category]];
    }
  },
  methods: {
    onCart() {
      this.cartDialog.items = [];
      this.cartDialog.total = 0;

      for (const [key, items] of Object.entries(this.products)) {
        key;

        for (const item of items) {
          if (item.quantity) {
            const toAdd = {
              name: item.name,
              image: item.image,
              quantity: item.quantity,
              price: item.quantity * item.price
            };

            this.cartDialog.total += toAdd.price;
            this.cartDialog.items.push(toAdd);
          }
        }
      }

      this.dialog = true;
    },
    addItem(item) {
      item.quantity += item.currentQuantity;
      this.cartSize += item.currentQuantity;

      item.currentQuantity = 1;
    },
    changeCategory(category) {
      for (const item of this.products[this.category]) {
        item.currentQuantity = 1;
      }

      this.category = category;
      this.search.model = "";
    },
    clearCart() {
      this.cartSize = 0;

      for (const [key, items] of Object.entries(this.products)) {
        key;
        for (const item of items) {
          item.quantity = 0;
        }
      }

      this.dialog = false;
    },
    showReviewDialog(url) {
      this.reviewDialog.show = true;
      this.reviewDialog.url = url;
    }
  }
};
</script>

<style>
.spin-input {
  width: 20px;
}
</style>