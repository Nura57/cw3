<template>
  <div class="container-fluid border-bottom mt-3">
    <div class="row justify-content-center">
      <div class="col-md-7">
        <h1 class="text-center page-caption">
          Search after school activities and lessons for your wards and kids 👨‍🎓
        </h1>
        <p class="text-center">
          Quality knowledge and skills to empower the new generation!
        </p>
      </div>
    </div>
    <div class="container">
      <div class="row mb-3 mt-2 justify-content-center">
        <div class="col-md-12">
          <div class="row justify-content-center">
            <div class="col-sm-5 mb-2">
              <input
                v-on:input="filterLessons"
                v-model="search_keyword"
                type="text"
                class="form-control rounded-0 p-2"
                placeholder="Search"
                aria-label="Recipient's Name"
                aria-describedby="button-addon2"
              />
            </div>
            <div class="col-sm-4 mb-2">
              <select
                v-on:change="sortList"
                v-model="attribute_sort"
                class="form-select p-2 rounded-0"
              >
                <option value="">Sort</option>
                <option value="title">Subject</option>
                <option value="location">Location</option>
                <option value="price">Price</option>
                <option value="spaces">Availability</option>
              </select>
            </div>
            <div class="col-sm-3 mb-2">
              <select
                v-on:change="sortList"
                v-model="sort_order"
                class="form-select p-2 rounded-0"
              >
                <option value="">Sort order</option>
                <option value="ascending">Ascending</option>
                <option value="descending">Descending</option>
              </select>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "heroComponent",
  props: {
    lesson: Object,
  },
};
</script>
