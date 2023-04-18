<template>
  <div class="container">
    <div class="row">
      <div class="col-12">
        <div id="contentTypesContainer">
          <div ref="tableTop" class="row events-container">
            <div class="col-lg-8">
              <div v-if="showPaginationTop && !showSelectedItem" class="my-1">
                <nav role="navigation" aria-label="Results data navigation">
                  <ul class="pagination d-flex flex-wrap mb-2 ms-0">
                    <li class="page-item" v-bind:class="{ disabled: currentPageIndex === 0 || loading }">
                      <button class="page-link" type="button" v-on:click="goToPage(currentPageIndex)">
                        Previous
                      </button>
                    </li>
                    <li v-if="showFirstLastBtns" class="page-item">
                      <button class="page-link" type="button" aria-label="Next" v-on:click="goToPage(1)">
                        <span aria-hidden="true">&laquo;</span>
                      </button>
                    </li>
                    
                    <li v-show="showPageBtns" v-for="pageBtn in pageBtns" class="page-item"
                      v-bind:class="{ disabled: currentPageIndex === pageBtn - 1 }" v-bind:key="pageBtn">
                      <button class="page-link" type="button" v-on:click="goToPage(pageBtn)">
                        {{ pageBtn }}
                      </button>
                    </li>
                    <li v-if="showFirstLastBtns" class="page-item">
                      <button class="page-link" type="button" aria-label="Next" v-on:click="goToPage(pageCount)">
                        <span aria-hidden="true">&raquo;</span>
                      </button>
                    </li>
                    <li class="page-item" v-bind:class="{ disabled: currentPageIndex + 1 === pageCount || loading }">
                      <button class="page-link" type="button" v-on:click="goToPage(currentPageIndex + 2)">
                        Next
                      </button>
                    </li>
                  </ul>
                </nav>
                <div v-if="loading == false" class="api-results-info">
                  <p>Total results: <strong>{{ totalCount }}</strong></p>
                  <p>
                    Current page:
                    <strong>{{ currentPageIndex+ 1 }}</strong>
                  </p>
                </div>
              </div>
              <div v-if="loading">
                <div class="spinner-grow" role="status"></div>
              </div>

              <!-- Events output -->
              <div v-show="!showSelectedItem" v-for="item in items" v-bind:key="item"
                class="ranger-event-card card card-item flex-md-row align-items-center">
                <div class="col-12 col-md-4 thumbnail-container d-flex justify-content-center px-2">
                  <img v-if="item.image != null" class="img-thumbnail
                                                               img-fluid mt-4 m-md-2"
                    v-bind:src="`https://www.cheshireeast.gov.uk/${item.image.asset.sys.uri}?width=225&height=150&fit=crop,center`"
                    v-bind:alt="item.title" />
                </div>
                <div class="card-body col-12 col-md-8 text-center text-md-start ps-md-4 ps-xl-2">
                  <button v-on:click="setSelectedItem(item)" role="button" class="ranger-event-title-link-button mb-3">
                    {{ item.entryTitle }}
                  </button>
                  <template v-if="formatDate(item.dateStartEnd.from) === formatDate(item.dateStartEnd.to)">
                    <p>{{ formatDate(item.dateStartEnd.from) }}.</p>
                    <p>
                      <strong>Time:</strong> {{ getTime(item.dateStartEnd.from) }} - {{
                        getTime(item.dateStartEnd.to)
                      }}.
                    </p>
                  </template>
                  <template v-else>
                    <p>From {{ getTime(item.dateStartEnd.from) }} on {{ formatDate(item.dateStartEnd.from) }}.</p>
                    <p>Until {{ getTime(item.dateStartEnd.to) }} on {{ formatDate(item.dateStartEnd.to) }}.</p>
                  </template>
                  <p>{{ item.excerpt }}</p>
                </div>
              </div>
            </div>
            <!-- Selected event details -->
            <div v-if="showSelectedItem" class="selectedItem">
              <button v-on:click="unSetSelectedItem" role="button" class="cec-button cec-button--back">
                Back to results
              </button>
              <h2 class="selected-item-details--title text-center">
                {{ selectedItem.title }}</h2>
              <template v-if="formatDate(selectedItem.dateStartEnd.from) === formatDate(selectedItem.dateStartEnd.to)">
                <p class="text-center fs-5">{{ formatDate(selectedItem.dateStartEnd.from) }}.</p>
              </template>
              <template v-else>
                <p class="text-center fs-5">
                  {{ formatDate(selectedItem.dateStartEnd.from) }} to
                  {{ formatDate(selectedItem.dateStartEnd.to) }}.
                </p>
              </template>
              <div class="selected-item-details">
                <div class="row">
                  <div class="col-12">
                    <img v-if="selectedItem.image != null" class="rounded
                       mx-auto d-block featured-img"
                      v-bind:src="'https://www.cheshireeast.gov.uk/' + 'selectedItem.image.asset.sys.uri'"
                      v-bind:alt="selectedItem.title" />
                    <hr />
                  </div>
                  <div class="col-lg-6 pb-lg-2">
                    <h3>Description</h3>
                    <div class="selected-item-details--description" v-html="selectedItem.description"></div>
                  </div>
                  <div class="col-lg-6 pb-2">
                    <h3>Details</h3>
                    <template
                      v-if="formatDate(selectedItem.dateStartEnd.from) === formatDate(selectedItem.dateStartEnd.to)">
                      <p>
                        <strong>Time:</strong> {{ getTime(selectedItem.dateStartEnd.from) }} - {{
                          getTime(selectedItem.dateStartEnd.to)
                        }}.
                      </p>
                    </template>
                    <template v-else>
                      <p><strong>From: </strong>{{ getTime(selectedItem.dateStartEnd.from) }}, {{
                        formatDate(selectedItem.dateStartEnd.from)
                      }}.</p>
                      <p><strong>To: </strong>{{ getTime(selectedItem.dateStartEnd.to) }}, {{
                        formatDate(selectedItem.dateStartEnd.to)
                      }}. </p>
                    </template>
                    <p>
                      <strong>Leader(s):</strong> {{ selectedItem.leaders }}
                    </p>
                    <p>
                      <strong>More information:</strong>
                      <span v-html="selectedItem.eventInformation"></span>
                    </p>
                    <p><strong>Tags:</strong></p>
                    <ul>
                      <li v-for="tag in selectedItem.tags" v-bind:key="tag">{{ tag.name }}</li>
                    </ul>
                    <h3>Meeting point</h3>
                    <p v-html="selectedItem.meetingPoint"></p>
                    <div class="selected-item-details__map">
                      <div id="map" ref="mapDiv">
                        <a target="_blank"
                          v-bind:href="'https://maps.google.com/maps?q=' + 'selectedItem.mapLocation.lat' + ',' + 'selectedItem.mapLocation.lon'"
                          class="cec-button cec-button-forward" role="button" aria-pressed="true">Get Directions
                          <small>(opens new window)</small></a>
                      </div>
                    </div>
                  </div>
                  <div class="col-12">
                    <hr />
                    <button v-on:click="unSetSelectedItem" role="button" class="cec-button cec-button--back">
                      Back to results
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <div v-if="!showSelectedItem" class="col-lg-4 mt-3 mt-lg-0">
              <div class="search-options-container border-secondary border border-1 rounded p-3">
                <h2 class="mt-0">Search for an event</h2>
                <div class="date-range">
                  <h3>Date range</h3>
                  <label for="start-date">From</label>
                  <input type="date" id="start-date" v-model="fromDate" min="2022-09-01" max="2025-12-31" />
                  <label for="end-date">To</label>
                  <input type="date" id="end-date" v-model="toDate" min="2022-09-01" max="2025-12-31"
                    v-on:change="searchFilter()" />
                </div>
                <div class="search-options mt-3">
                  <div v-if="showSearch" class="input-group content-type-search">
                    <input @input="searchFilter" @focus="clearAlert" autocomplete="off" type="text"
                      class="form-control me-2 border-secondary border border-1" placeholder="Type here..."
                      v-model="searchTerm" aria-label="Search term" id="contentTypeSearchInput" />
                    <div class="input-group-append">
                      <button type="button" class="btn btn-outline-secondary" v-on:click="searchFilter"
                        id="contentTypeSearchBtn">
                        Search
                      </button>
                    </div>
                    <button class="btn btn-outline-secondary w-100 mt-3" type="button" v-on:click="resetSearch">
                      Clear
                    </button>
                  </div>
                  <div v-if="showSearch" class="search-error-messages">
                    <div v-if="searchAlert" class="alert alert-secondary mt-2" role="alert">
                      Please enter a search term.
                    </div>
                  </div>
                </div>
              </div>
              <div class="category-options pt=5 mt-5">
                <h3 class="mb-3">Or filter events by category</h3>
                <div class="form-check" v-for="category in categories" v-bind:key="category">
                  <input class="form-check-input" type="checkbox" v-bind:id="category" v-bind:ref="category"
                    v-on:change="setFilter(category)" />
                  <label class="form-check-label" v-bind:for="category">{{ category }}</label>
                </div>
              </div>
            </div>
          </div>

          <!-- Bottom navigation -->
          <div v-if="showPaginationBottom && !showSelectedItem" class="mt-1 mb-1">
            <nav aria-label="Results data navigation">
              <ul class="pagination mb-0">
                <li class="page-item" v-bind:class="{ disabled: currentPageIndex === 0 }">
                  <a class="page-link" v-on:click="goToPage(currentPageIndex)">Previous</a>
                </li>
                <li v-if="showFirstLastBtns" class="page-item">
                  <a class="page-link" href="#" aria-label="Next" v-on:click="goToPage(1)">
                    <span aria-hidden="true">&laquo;</span>
                  </a>
                </li>
                <li v-show="showPageBtns" v-for="pageBtn in pageBtns" class="page-item"
                  v-bind:class="{ disabled: currentPageIndex === pageBtn - 1 }" v-bind:key="pageBtn">
                  <a class="page-link" v-on:click="goToPage(pageBtn)">{{ pageBtn }}</a>
                </li>
                <li v-if="showFirstLastBtns" class="page-item">
                  <a class="page-link" href="#" aria-label="Next" v-on:click="goToPage(pageCount)">
                    <span aria-hidden="true">&raquo;</span>
                  </a>
                </li>
                <li class="page-item" v-bind:class="{ disabled: currentPageIndex + 1 == pageCount }">
                  <a class="page-link" v-on:click="goToPage(currentPageIndex + 2)">Next</a>
                </li>
              </ul>
            </nav>
            <div v-if="loading == false" class="api-results-info">
              <p>Total results: <strong>{{ totalCount }}</strong></p>
              <p>Current page: <strong>{{ currentPageIndex+ 1 }}</strong></p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'RangerEvents',
  data() {
    let contentType = "rangerEvents";
    let projectId = "website";
    return {
      accessToken: "QCpZfwnsgnQsyHHB3ID5isS43cZnthj6YoSPtemxFGtcH15I", // API access token, replace if expired
      linkDepth: 5, // leave at 5 if unsure.
      pageSize: 20,
      showPageBtns: true,
      showFirstLastBtns: false,
      showPaginationTop: true,
      showPaginationBottom: true,
      showSearch: true,
      url:
        `https://cms-chesheast.cloud.contensis.com/api/delivery/projects/Website/contentTypes/${contentType}/entries/`,
      searchFields: ["title", "description"],
      numOfPaginationBtns: 0, // Set to 0 to show all.
      includePageCount: true, // Set whether to request the page count parameter for the API calls.
      items: [],
      loading: true,
      currentPageIndex: 0,
      totalCount: null,
      pageCount: null,
      pageBtns: [],
      contentType: contentType,
      searchTerm: "",
      title: "",
      description: "",
      searchAlert: false,
      fromDate: "",
      toDate: "",
      filterField: "tags",
      categories: ["Bring a packed lunch",
        "Bring binoculars if you have them",
        "Car parking charge",
        "Children must be accompanied by an adult",
        "Easy walking grade",
        "Full accessibility - level or ramped access. Accessible toilets available",
        "Healthy walk / Event",
        "Historial walk / Event",
        "Ideal for families and accompanied children",
        "Moderate walking grade",
        "Partial accessibility - level or ramped access",
        "Partnership event",
        "Places limited - please book in advance",
        "Please leave your dog at home",
        "Please wear suitable boots and clothing",
        "Refreshment shop, opportunity for refreshments",
        "Strenuous walking grade",
        "There is a charge for this event",
        "Wildlife walk / Event"],
      categoriesChecked: [],
      firstRender: true,
      selectedItem: null,
      showSelectedItem: false,
      dateOptions: {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
      },
      rxDate: /^\d{4}-\d{2}-\d{2}(?:T\d{2}:\d{2}:\d{2})?(?:\.\d*)?Z?$/,
    };
  },
  filters: {
    currencydecimal(value) {
      return value.toFixed(2);
    },
  },
  methods: {
    prependUrl: function (value) {
      return `${location.protocol}//${location.host}${location.pathname}?event=${value}`;
    },
    clearAlert: function () {
      this.searchAlert = false;
    },
    searchFilter: function () {
      let fromDate = (this.fromDate.length > 0) ? new Date(this.fromDate) : false;
      let toDate = (this.toDate.length > 0) ? new Date(this.toDate) : false;
      this.searchedItems = this.filteredItems.filter(item =>
        this.searchFields.some((term) => {
          return (!this.searchTerm || item[term].toLowerCase().includes(this.searchTerm.toLowerCase())) &&
            ((!fromDate || item.dateStartEnd.to > fromDate) && (!toDate || item.dateStartEnd.to < toDate));
        }));
      if (this.searchedItems.length > 0) {
        this.searchedItems.sort(this.sortObjDate());
      }
      this.calculatePages();
    },
    setFilter: function (cat) {
      const index = this.categoriesChecked.indexOf(cat);
      if (index > -1) {
        this.categoriesChecked.splice(index, 1);
      } else {
        this.categoriesChecked.push(cat);
      }
      this.filterByCategories();
      this.searchFilter();
    },
    filterByCategories: function () {
      this.filteredItems = [];
      if (this.categoriesChecked.length === 0) {
        this.filteredItems = this.copyItems.slice();
      } else {
        this.filteredItems = this.copyItems.filter((elem) =>
          elem[this.filterField].map(a => a.name).some((cat) =>
            this.categoriesChecked.includes(cat)));
      }
      this.searchFilter();
    },
    createDates: function (arr) {
      return arr.map((e) => {
        return Object.fromEntries(
          Object.entries(e).map(([k, v]) => {
            if (k.toLowerCase().includes("date")) {
              if (typeof v === "string" && v.match(this.rxDate)) {
                return [k, new Date(v)];
              } else if (typeof v === "object") {
                try {
                  return [k, { 'from': new Date(v["from"]), 'to': new Date(v["to"]) }];
                }
                catch (err) {
                  return [k, v];
                }
              }
            } else {
              return [k, v];
            }
          })
        );
      });
    },
    formatDate: function (value) {
      return value.toLocaleString('en-GB', this.dateOptions);
    },
    getTime: function (value) {
      let time = value.toLocaleTimeString([], { hour: "numeric", minute: "2-digit", hour12: true });
      if (time === "0:00 pm") {
        return "12 noon";
      }
      else if (time.startsWith("0")) {
        time = `12${time.slice(1)}`;
      }
      return time.replace(' ', '');
    },
    getBaseUrl: function () {
      return `${this.url}?accessToken=${this.accessToken}&linkDepth=${this.linkDepth}&pageSize=500`;
    },
    goToPage: function (pageNum) {
      this.$refs.tableTop.scrollIntoView();
      this.items = this.pages[pageNum - 1];
      this.currentPageIndex = pageNum - 1;
      this.lastSearch = this.searchTerm;
    },
    sortObjDate: function () {
      return (a, b) => {
        return a.dateStartEnd.from - b.dateStartEnd.from;
      };
    },
    calculatePages: function () {
      this.totalCount = this.searchedItems.length;
      this.pageCount = Math.ceil(this.totalCount / this.pageSize);
      this.currentPageIndex = 0;
      this.pageBtns = Array.from({ length: this.pageCount }, (_, i) => i + 1);
      this.createPages();
      this.items = this.pages[0];
    },
    createPages: function () {
      this.pages = [
        ...Array(Math.ceil(this.searchedItems.length / this.pageSize)),
      ].map((_) => this.searchedItems.splice(0, this.pageSize));
    },
    callAxios: function (url, slug = null) {
      console.log(url);
      axios
        .get(url)
        .then((response) => {
          this.copyItems = this.createDates(response.data.items);

          if (this.copyItems.length > 0) {
            this.copyItems.sort(this.sortObjDate());
          }
          this.filteredItems = this.copyItems.slice();
          this.searchedItems = this.copyItems.slice();
          vanillaToast.show("Results updated.");
          this.loading = false;
          this.calculatePages();

          if (slug !== null) {
            this.setSelectedItem(this.copyItems.find(elem => elem.sys.slug === slug));
          }
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => this.loading = false);
    },

    resetSearch: function () {
      this.searchTerm = "";
      this.fromDate = "";
      this.toDate = "";
      this.searchedItems = this.filteredItems.slice();
      this.calculatePages();
    },
    setSelectedItem: function (item) {
      window.scrollTo(0, 0);
      this.selectedItem = item;
      document.title = item.entryTitle;
      this.showSelectedItem = true;
      (new URLSearchParams(window.location.search)).delete("event");
      window.history.pushState(
        "",
        "",
        this.prependUrl(item.sys.slug));
    },
    unSetSelectedItem: function () {
      window.scrollTo(0, 0);
      this.selectedItem = null;
      document.title = this.title;
      this.showSelectedItem = false;
      window.history.pushState({}, "", document.location.href.split("?")[0]);
    },
    getShareableLink(selectedItem) {
      (new URLSearchParams(window.location.search)).delete("event");
      navigator.clipboard.writeText(this.prependUrl(selectedItem.sys.slug));
      vanillaToast.show("Event URL copied to clipboard.");
    },
    back: function () {
      location.reload();
    }
  },
  mounted() {
    this.title = document.title;
    const params = new Proxy(new URLSearchParams(window.location.search), {
      get: (searchParams, prop) => searchParams.get(prop),
    });

    if (params.event !== null) {
      let url = window.location.href.toLowerCase().includes("preview")
        ? `${this.getBaseUrl()}&versionStatus=latest`
        : this.getBaseUrl();
      this.callAxios(url, params.event);
    } else {
      this.callAxios(this.getBaseUrl());
    }

    if (this.showSearch) {
      document
        .getElementById("contentTypeSearchInput")
        .addEventListener("keyup", function (event) {
          event.preventDefault();
          if (event.keyCode === 13) {
            document.getElementById("contentTypeSearchBtn").click();
          }
        });
      window.addEventListener(
        "keydown",
        function (e) {
          if (
            e.keyIdentifier == "U+000A" ||
            e.keyIdentifier == "Enter" ||
            e.keyCode == 13
          ) {
            if (
              e.target.nodeName == "INPUT" &&
              e.target.type == "text"
            ) {
              e.preventDefault();
              return false;
            }
          }
        },
        true
      );
    }
  }
}
</script>
