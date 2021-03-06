<template>
  <div id="app">
    <div id="wrapper">
      <div id="data">
        <div>
          <div class="logo-wrapper">
            <img class="logo" src="./assets/logo.png" />
            <h1>Fire Overlay</h1>
            <p>Overlay wildfires to get a feeling about the size of the destruction.</p>
          </div>
        </div>

        <div class="selectors">
          <div class="selector selector--fire">
            <h2>Fire:</h2>
            <v-select
              value="2019 Australian Bushfires"
              :clearable="false"
              :options="['2019 Australian Bushfires']"
            ></v-select>
          </div>
          <div class="selector selector--city">
            <h2>City:</h2>
            <v-select
              :value="selectedCity"
              @input="setSelectedCity"
              @search="onSearchCity"
              :clearable="false"
              label="name"
              :options="cities"
            ></v-select>
          </div>
        </div>

        <div class="notice">
          <p style="color:red">
            The red circle designates the area actively burned down.
            <span
              style="color:gray"
            >The gray circle designates the area affected by the smoke.</span>
          </p>
        </div>

        <div class="contribute">
          <h3>Every dollar counts!</h3>
          <p>
            Currently the donations are over
            <strong>140 mil. $</strong>
          </p>
          <div class="contribute-links">
            <a href="https://fundraise.redcross.org.au/drr">
              Help with
              <strong>RedCross</strong>
            </a>
            <a href="https://www.wwf.org.au/get-involved/bushfire-emergency">
              Help with
              <strong>WWF</strong>
            </a>
          </div>
        </div>

        <div class="author">
          <small>
            Made by
            <a target="_blank" href="http://github.com/gokaykucuk/">gokaykucuk</a> &
            <a target="_blank" href="http://fatihgozenc.com">fatihgozenc</a>
          </small>
        </div>
      </div>
      <div id="map">
        <Map ref="mainMap" :lat="lat" :lng="lng"></Map>
      </div>
    </div>
  </div>
</template>

<script>
import "./scss/app.scss";

import "./utils/constants";
import Map from "./components/Map.vue";
import { Client } from "faunadb";
import { constants } from "buffer";
import _ from "lodash";
const faunadb = require("faunadb");
const q = faunadb.query;

export default {
  name: "app",
  components: {
    Map
  },
  data() {
    return {
      lat: 0,
      lng: 0,
      cities: [],
      selectedCity: null
    };
  },
  async mounted() {
    const client = new faunadb.Client({
      secret: FAUNADB_CLIENT_KEY
    });
    const citiesResponse = await client.query(
      q.Map(
        q.Paginate(q.Match(q.Index("all_cities"))),
        q.Lambda("X", q.Get(q.Var("X")))
      )
    );
    this.cities = citiesResponse.data.map(city => city.data);
    this.selectedCity = _.sample(this.cities);
    this.lat = this.selectedCity.lat;
    this.lng = this.selectedCity.lng;
    this.updateLocation();
  },
  methods: {
    setSelectedCity: function(newCity) {
      this.selectedCity = newCity;
      this.updateLocation();
    },
    onSearchCity: function(searchString, loading) {
      if (!(searchString.length > 0 && searchString.length < 5)) {
        return;
      }
      loading(true);
      this.searchCities(loading, searchString, this);
    },
    updateLocation: function() {
      this.$refs.mainMap.updateSelection(
        this.selectedCity.lat,
        this.selectedCity.lng,
        63000
      );
    },
    searchCities: _.debounce(async (loading, searchParam, vm) => {
      const client = new faunadb.Client({
        secret: FAUNADB_CLIENT_KEY
      });

      const citiesResponse = await client.query(
        q.Map(
          q.Paginate(
            q.Match(
              q.Index(`cities_autocomplete_${searchParam.length}`),
              q.LowerCase(searchParam)
            )
          ),
          q.Lambda("X", q.Get(q.Var("X")))
        )
      );

      vm.cities = citiesResponse.data.map(city => city.data);
      loading(false);
    }, 350)
  }
};
</script>