<template>
  <div class="selectedItem">
    <h1>Slug: {{ slug }}</h1>

    <button v-on:click="unSetSelectedItem" role="button" class="cec-button cec-button--back">
      Back to results
    </button>
    <template v-if="selectedItem != null">
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
            <template v-if="formatDate(selectedItem.dateStartEnd.from) === formatDate(selectedItem.dateStartEnd.to)">
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
              <li v-for="tag in selectedItem.tags" :key="tag">{{ tag.name }}</li>
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
    </template>

    <template v-if="selectedItem != null">
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
            <template v-if="formatDate(selectedItem.dateStartEnd.from) === formatDate(selectedItem.dateStartEnd.to)">
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
              <li v-for="tag in selectedItem.tags" :key="tag">{{ tag.name }}</li>
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
    </template>

  </div>

</template>
  
<script>
export default {
  name: 'RangerEvents',
  data() {
    return {
      selectedItem: null,
      test: "a"
    }
  },
  async asyncData({ params }) {
    const slug = params.slug // When calling /abc the slug will be "abc"
    return { slug }
  },
  methods: {

  },
  /* async fetch() {
    this.selectedItem = await fetch('https://cms-chesheast.cloud.contensis.com/api/delivery/projects/Website/contentTypes/rangerEvents/entries/').then(res =>
      res.json()
    )
  }, */

  mounted() {
    // Get slug
    // Perform axios call
    this.test = "b"
  }
}
</script>
