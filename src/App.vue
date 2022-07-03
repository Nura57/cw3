<template>
  <div id="app">
    <nav class="navbar navbar-light bg-white border-bottom navbar-expand">
      <div class="container-fluid">
        <a class="navbar-brand fw-bold" href="#">After-school Lessons 📚</a>
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarNav"
          aria-controls="navbarNav"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav ms-auto">
            <li class="nav-item">
              <button
                v-bind:disabled="hideCart()"
                v-on:click="toggleCheckOut"
                type="button"
                class="btn rounded-10 btn-md btn-warning position-relative"
              >
                <i class="fas fa-cart-arrow-down"></i>
                <span
                  class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-dark"
                >
                  {{ countCart() }}
                </span>
              </button>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <div class="container mt-3">
      <hero-component></hero-component>
      <cart-component
        v-if="showCheckOut"
        :cartItems="cartItemsInfo()"
        v-on:remove-lesson="removeFromCart($event)"
        v-on:checkout="checkOut($event)"
      ></cart-component>
      <div v-else class="row">
        <div class="col-sm-3 mb-2" v-for="lesson in lessons" :key="lesson.id">
          <lesson-component
            :lesson="lesson"
            v-on:add-to-cart="addToCart($event)"
          ></lesson-component>
        </div>
      </div>
    </div>

    <footer>
      <div class="container mt-5">
        <div class="rown justify-content-center">
          <p class="text-center m-0">Haske Corp | &copy; 2022</p>
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
import heroComponent from "./components/heroComponent.vue";
import lessonComponent from "./components/lessonComponent.vue";
import cartComponent from "./components/cartComponent.vue";
export default {
  name: "App",
  components: {
    heroComponent,
    lessonComponent,
    cartComponent,
  },
  data() {
    return {
      lessons: [],
      cart_items: [],
      search_keyword: "",
      sort_order: "",
      attribute_sort: "",
      showCheckOut: false,
    };
  },
  created() {
    this.fetchLessons();
  },
  methods: {
    // Toggle checkout
    toggleCheckOut: function () {
      if (this.showCheckOut == false) {
        this.showCheckOut = true;
      } else this.showCheckOut = false;
    },

    addToCart(lesson) {
      if (lesson.spaces >= 1) {
        let lessonInCart = false;
        if (this.countCart() >= 1) {
          for (let i = 0; i < this.cart_items.length; i++) {
            if (this.cart_items[i].id == lesson._id) {
              this.cart_items[i].spaces += 1;
              lessonInCart = true;
              break;
            }
          }
          if (lessonInCart == false) {
            let item = {};
            item.id = lesson._id;
            item.spaces = 1;
            this.cart_items.push(item);
          }
        } else {
          let item = {};
          item.id = lesson._id;
          item.spaces = 1;
          this.cart_items.push(item);
        }
        lesson.spaces -= 1;
      } else {
        lesson.spaces = 0;
      }
    },

    countCart() {
      return this.cart_items.length;
    },

    cartItemsInfo() {
      let cart_items_info = [];
      for (let i = 0; i < this.cart_items.length; i++) {
        for (let j = 0; j < this.lessons.length; j++) {
          if (this.lessons[j]._id == this.cart_items[i].id) {
            let item = {};
            item.id = this.cart_items[i].id;
            item.title = this.lessons[j].title;
            item.location = this.lessons[j].location;
            item.price = this.lessons[j].price;
            item.url = this.lessons[j].url;
            item.spaces = this.cart_items[i].spaces;
            cart_items_info.push(item);
          }
        }
      }
      return cart_items_info;
    },

    hideCart() {
      if (this.countCart() >= 1) {
        return false;
      }
      return true;
    },

    removeFromCart(obj) {
      for (let i = 0; i < this.lessons.length; i++) {
        if (this.lessons[i]._id == obj.id) {
          this.lessons[i].spaces += obj.spaces;
        }
      }
      for (let i = 0; i < this.cart_items.length; i++) {
        if (this.cart_items[i].id == obj.id) {
          this.cart_items.splice(i, 1);
        }
      }
    },

    checkOut(order_details) {
      let order = {
        name: order_details.firstname + " " + order_details.lastname,
        phone_number: order_details.mobile,
        items: this.cart_items,
      };
      let order_string = JSON.stringify(order);
      fetch("https://coursework-two-king.herokuapp.com/collection/orders", {
        method: "POST",
        body: order_string,
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      })
        .then((response) => response.json())
        .then((json_response) => {
          console.log(json_response);
          this.updateSpaces();
        })
        .catch((err) => console.log(err));
    },

    updateSpaces() {
      let spaces_upd = [];
      for (let i = 0; i < this.cart_items.length; i++) {
        for (let j = 0; j < this.lessons.length; j++) {
          if (this.cart_items[i].id == this.lessons[j]._id) {
            let item = {
              id: this.cart_items[i].id,
              spaces: this.lessons[j].spaces,
            };
            spaces_upd.push(item);
          }
        }
      }
      let spaces_upd_string = JSON.stringify(spaces_upd);
      fetch("https://cst3145-cw2-1.herokuapp.com/collection/activities", {
        method: "PUT",
        body: spaces_upd_string,
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      })
        .then((response) => response.json())
        .then((response) => {
          console.log(response);
          this.cart_items = [];
          alert("Order submitted successfully!");
          this.showCheckOut = false;
        })
        .catch((err) => console.log(err));
    },

    fetchLessons() {
      fetch(`https://cst3145-cw2-1.herokuapp.com/collection/activities`)
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          this.lessons = data;
          console.log(this.lessons);
        })
        .catch((err) => {
          this.lessons = [];
          console.log(`unable to get lessons: ${err}`);
        });
    },

    filterLessons() {
      fetch(
        `https://cst3145-cw2-1.herokuapp.com/collection/activities/search?filter=${this.search_keyword}`
      )
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          this.lessons = data;
        })
        .catch((err) => {
          this.lessons = [];
          console.log(`unable to get lessons: ${err}`);
        });
    },

    sortList() {
      if (this.attribute_sort == "location") {
        this.lessons.sort(this.sortbyLocation);
      } else if (this.attribute_sort == "price") {
        this.lessons.sort(this.sortbyPrice);
      } else if (this.attribute_sort == "spaces") {
        this.lessons.sort(this.sortbySpaces);
      } else {
        this.lessons.sort(this.sortbyTitle);
      }

      if (this.sort_order == "descending") {
        this.lessons.reverse();
      }
    },

    sortbyTitle(a, b) {
      if (a.title.toLowerCase() < b.title.toLowerCase()) {
        return -1;
      }
      if (a.title.toLowerCase() > b.title.toLowerCase()) {
        return 1;
      }
      return 0;
    },

    sortbyLocation(a, b) {
      if (a.location.toLowerCase() < b.location.toLowerCase()) {
        return -1;
      }
      if (a.location.toLowerCase() > b.location.toLowerCase()) {
        return 1;
      }
      return 0;
    },

    sortbyPrice(a, b) {
      return a.price - b.price;
    },

    sortbySpaces(a, b) {
      return a.spaces - b.spaces;
    },
  },
};
</script>
