<template>
  <div class="main">
    <div class="flex">
      <div class="map-holder">
        <div id="map"></div>
      </div>

      <div class="dislpay-arena">
        <div class="coordinates-header">
          <h3>Current Coordinates</h3>
          <p>Latitude: {{ center[0] }}</p>
          <p>Longitude: {{ center[1] }}</p>
        </div>

        <div class="coordinates-header">
          <h3>Current Location</h3>

          <div class="form-group">
            <input
                type="text"
                class="location-control"
                :value="location"
                readonly
            />
            <button type="button" class="copy-btn" @click="copyLocation">
              Copy
            </button>
          </div>

          <button
              type="button"
              :disabled="loading"
              :class="{ disabled: loading}"
              class="location-btn"
              @click="getLocation"
          >
            Get Location
          </button>
          <div>
            <button
                type="button"
                :disabled="loading"
                :class="{ disabled: loading }"
                class="location-btn"
                @click="showModal"
            >
              Add plot
            </button>
          </div>
          <PlotForm
              v-show="isModalVisible"
              @close="closeModal"
              @setplots="setPlots"
              @createmap="createMap"
              :getPlots="getPlots"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import axios from "axios";
import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";
import PlotForm from "@/components/PlotForm.vue";

export default {
  components: {
    PlotForm,
  },
  data() {
    return {
      loading: false,
      location: "",
      access_token: "pk.eyJ1Ijoia2traWlra2siLCJhIjoiY2xudWZvd2FqMGMzczJrdGFraG81dDJ1ayJ9.MPEcH3wbwpCsBDvswpmoaA",
      center: [0, 0],
      map: null,
      isModalVisible: false,
      newPlot: {
        level: null,
        points: [],
      },
      plots: null
    };
  },

  mounted() {
    this.getPlots().then().catch();
    this.createMap();
  },

  methods: {
    setPlots(plots) {
      console.log('setPlots')
      this.plots = plots
    },
    async getPlots() {
      const res = await axios.get(`http://localhost:1337/api/plots`, {
        headers: {
          Authorization: `Bearer 3c6fe90c9d56419452691a3f05b6a818e300b953508755f21e7ed99711668d7c525a4300a7354f5c8646c96b7a8349a9975cce6392d0e7b3ec758ea64729bd012a8cc320516ffd23f33d118cc72e73af5712f7394653da3ba7d4e568c8c857f4d1cc6d87ab9155181a96b4c370abf68dd31ddf239e27cdeaed16d48c8dbf0662`
        }
      });

      this.plots = res.data['data']

      return res.data['data']
    },
    async createMap() {
      try {
        this.map ? this.map.remove() : null
        mapboxgl.accessToken = this.access_token;
        this.map = new mapboxgl.Map({
          container: "map",
          style: "mapbox://styles/mapbox/streets-v11",
          center: this.center,
          zoom: 11,
        });
        await this.getPlots()
        this.map.on('load', () => {
          for (const  { id, attributes: { points } } of this.plots) {
            console.log(id)
            this.map.addSource(String(id), {
              'type': 'geojson',
              'data': {
                'type': 'Feature',
                'geometry': {
                  'type': 'Polygon',
                  'coordinates': [
                    points
                  ]
                }
              }
            });

            this.map.addLayer({
              'id': String(id),
              'type': 'fill',
              'source': String(id),
              'layout': {},
              'paint': {
                'fill-color': '#0080ff',
                'fill-opacity': 0.5
              }
            });
            this.map.addLayer({
              'id': String(id) + 'outline',
              'type': 'line',
              'source': String(id),
              'layout': {},
              'paint': {
                'line-color': '#000',
                'line-width': 3
              }
            })
          }
        })
        let geocoder =  new MapboxGeocoder({
          accessToken: this.access_token,
          mapboxgl: mapboxgl,
          marker: false,
        });

        this.map.addControl(geocoder);

        geocoder.on("result", (e) => {
          const marker = new mapboxgl.Marker({
            draggable: true,
            color: "#D80739",
          })
              .setLngLat(e.result.center)
              .addTo(this.map);
          this.center = e.result.center;

          marker.on("dragend", (e) => {
            this.center = Object.values(e.target.getLngLat());
          });
        });
      } catch (err) {
        console.log("map error", err);
      }
    },
    showModal() {
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    async getLocation() {
      try {
        this.loading = true;
        const response = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${this.center[0]},${this.center[1]}.json?access_token=${this.access_token}`
        );

        this.loading = false;
        this.location = response.data.features[0].place_name;
      } catch (err) {
        this.loading = false;
        console.log(err);
      }
    },
    copyLocation() {
      if (this.location) {
        navigator.clipboard.writeText(this.location);
        alert("Location Copied")
      }
      return;
    },
  },
};
</script>

<style scoped>
.main {
  padding: 45px 50px;
}
.flex {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

.map-holder {
  width: 65%;
}
#map {
  height: 70vh;
}
.dislpay-arena {
  background: #ffffff;
  box-shadow: 0px -3px 10px rgba(0, 58, 78, 0.1);
  border-radius: 5px;
  padding: 20px 30px;
  width: 25%;
}
.coordinates-header {
  margin-bottom: 50px;
}
.coordinates-header h3 {
  color: #1f2a53;
  font-weight: 600;
}

.coordinates-header p {
  color: rgba(13, 16, 27, 0.75);
  font-weight: 600;
  font-size: 0.875rem;
}
.form-group {
  position: relative;
}
.location-control {
  height: 30px;
  background: #ffffff;
  border: 1px solid rgba(31, 42, 83, 0.25);
  box-shadow: 0px 0px 10px rgba(73, 165, 198, 0.1);
  border-radius: 4px;
  padding: 0px 10px;
  width: 90%;
}
.location-control:focus {
  outline: none;
}
.location-btn {
  margin-top: 15px;
  padding: 10px 15px;
  background: #d80739;
  box-shadow: 0px 4px 10px rgba(73, 165, 198, 0.1);
  border-radius: 5px;
  border: 0;
  cursor: pointer;
  color: #ffffff;
  font-size: 0.875rem;
  font-weight: 600;
}

.location-btn {
  margin-top: 15px;
  padding: 10px 15px;
  background: brown;
  box-shadow: 0px 4px 10px rgba(73, 165, 198, 0.1);
  border-radius: 5px;
  border: 0;
  cursor: pointer;
  color: #ffffff;
  font-size: 0.875rem;
  font-weight: 600;
}

.location-btn:focus {
  outline: none;
}
.disabled {
  background: #db7990;
  cursor: not-allowed;
}
.copy-btn {
  background: #f4f6f8 0% 0% no-repeat padding-box;
  border: 1px solid #f4f6f8;
  border-radius: 0px 3px 3px 0px;
  position: absolute;
  color: #5171ef;
  font-size: 0.875rem;
  font-weight: 500;
  height: 30px;
  padding: 0px 10px;
  cursor: pointer;
  right: 3.5%;
  top: 5%;
}
.copy-btn:focus {
  outline: none;
}
</style>
