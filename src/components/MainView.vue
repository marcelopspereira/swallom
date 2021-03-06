<template lang="html">
  <div class="layer-wrapper">
    <div class="map-layer">
      <gmap-map ref="mainmap"
        :center="position"
        :zoom="zoom"
        :options="mapOptions"
        map-type-id="terrain"
        style="width: 100vw; height: 92vh"
      >
        <gmap-marker :position="position" icon="/static/img/streetview-icon.png" v-if="$refs && $refs.mainmap"></gmap-marker>
        <gmap-circle :center="position" :radius="radius" options:="circleOptions" v-if="$refs && $refs.mainmap"></gmap-circle>
      </gmap-map>
    </div>
    <div v-if="!$refs || !$refs.mainmap" class="map-layer loading">
      <v-progress-circular indeterminate v-bind:size="70" v-bind:width="7" class="pink--text lighten-2"></v-progress-circular>
    </div>
    <div class="control-layer">
      <div  class="top-con">
        <v-chip v-for="(type, i) in alltypes" :key="i" @click="toggleType(i)" :outline="!type.selected" class="secondary secondary--text" :class="{'white--text': type.selected}">{{type.name}}</v-chip>
      </div>
      <v-layout column class="bottom-con">
        <v-flex>
          <v-slider v-model="radius" :hint="'radius: '+radius+'m'" :persistent-hint="true" min="200" max="2000" ></v-slider>
        </v-flex>
        <v-flex>
          <v-btn round primary dark block class="pink lighten-2" @click="random" :disabled="type.length < 1">
            <span v-show="!searching">Random</span>
            <v-progress-circular indeterminate class="white--text" :size="20" v-show="searching"></v-progress-circular>
          </v-btn>
        </v-flex>
      </v-layout>
    </div>
    <v-snackbar error v-model="error.status">
      {{ error.message }}
      <v-btn dark flat @click.native="error.status = false">Close</v-btn>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      position: {lat: 0.0, lng: 0.0},
      radius: 800,
      type: ['restaurant'],
      mapOptions: {
        zoomControl: false,
        mapTypeControl: false,
        scaleControl: true,
        streetViewControl: false,
        rotateControl: false,
        fullscreenControl: false,
      },
      circleOptions: {
        strokeColor: '#FF0000',
        strokeOpacity: 0.6,
        strokeWeight: 1,
        fillColor: '#FF0000',
        fillOpacity: 0.2,
      },
      alltypes: [
        { name: 'Cafe', value: 'cafe', selected: false },
        { name: 'Bakery', value: 'bakery', selected: false },
        { name: 'Bar', value: 'bar', selected: false },
        { name: 'Convenience Store', value: 'convenience_store', selected: false },
        { name: 'Department Store', value: 'department_store', selected: false },
        { name: 'Restaurant', value: 'restaurant', selected: true }
      ],
      error: {
        status: false,
        message: ''
      },
      searching: false
    }
  },
  computed: {
    resultList: function() {
      return this.$store.state.resultList;
    },
    zoom: function() {
      return 15 - Math.round((this.radius-800)*0.002);
    }
  },
  mounted: function() {
    var vm = this;

    this.searching = false;

    // Get device location
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(function(position) {
        vm.position = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        };
      }, function() {
        console.log("Location Error")

        this.error.message = 'Sad, Location Error';
        this.error.status = true;
      });
    } else {
      console.log("Browser doesn't support Geolocation");
      this.error.message = 'Sad, Your browser doesn\'t support Geolocation';
      this.error.status = true;
    }
  },
  methods: {
    toggleType : function(i) {
      if(this.alltypes[i].selected) {
        //toggle off
        this.type.splice(this.type.indexOf(this.alltypes[i].value),1);
        this.alltypes[i].selected = false;
      } else {
        //toggle on
        this.type.push(this.alltypes[i].value);
        this.alltypes[i].selected = true;
      }
    },
    random : function() {
      this.searching = true;

      //Init place service
      var request = {
        location: this.position,
        radius: this.radius,
        type: this.type
      };

      let service = new google.maps.places.PlacesService(this.$refs.mainmap.$mapObject);

      var vm = this;

      //Get place list
      service.nearbySearch(request, function(results, status) {
        if (status == google.maps.places.PlacesServiceStatus.OK && results.length > 0) {

          // store
          vm.$store.commit('updateResult',results);

          // route
          vm.$router.push('/result/'+vm.position.lat+'/'+vm.position.lng+'/'+vm.zoom+'/');
        } else {
          vm.searching = false;
          vm.error.message = 'Sad, nothing found :(';
          vm.error.status = true;
        }
      });
    }
  }
}

</script>

<style lang="css">
  .layer-wrapper {
    position: relative;
  }

  .map-layer {
    position: absolute;
    height: 92vh;
  }

  .control-layer {
    position: relative;
    width: 100vw;
    height: 92vh;
  }

  .top-con {
    position: absolute;
    top: 0;
    width: 100vw;
    padding: 3vw 5vw;
  }

  .bottom-con {
    position: absolute;
    bottom: 0;
    width: 100vw;
    padding: 5vw;
  }

  .loading {
    width: 100vw;
    background-color: white;
    opacity: 0.6;
    z-index: 999;
    padding-top: 40vh;
  }
</style>
