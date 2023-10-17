<template>
  <div class="modal-backdrop">
    <div class="modal">
      <header class="modal-header">
        <button
            type="button"
            class="btn-close"
            @click="close"
        >
          x
        </button>
      </header>

      <section class="modal-body">
        <form @submit.prevent="addPlot">
          <label for="level">Level:</label>
          <input type="number" id="level" v-model="newPlot.level" required>
          <div>
            <label for="points">Coordinates (Lat, Long):</label>
          </div>
          <div key="1">
            <label for="point1">1 Point:</label>
            <input id ="point1" type="text" v-model="newPlot.points[0]" required>
          </div>
          <div key="2">
            <label for="point2">2 Point:</label>
            <input id ="point2" type="text" v-model="newPlot.points[1]" required>
          </div>
          <div key="3">
            <label for="point3">3 Point:</label>
            <input id ="point3" type="text" v-model="newPlot.points[2]" required>
          </div>
          <div key="4">
            <label for="point4">4 Point:</label>
            <input id ="point4" type="text" v-model="newPlot.points[3]" required>
          </div>

          <button type="submit" @click="addNewPlot">Add Plot</button>
        </form>
      </section>

      <footer class="modal-footer">
        <button
            type="button"
            class="btn-green"
            @click="close"
        >
          Close Modal
        </button>
      </footer>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  props: {
    getPlots: Function,
  },
  name: 'PlotForm',
  data() {
    return {
      newPlot: {
        level: null,
        points: [],
      },
    };
  },
  methods: {
    close() {
      this.$emit('close');
    },

    async addNewPlot() {
      const points = this.newPlot.points.map((el) => {
        const arr = el.split(" ")
        arr[0] = Number(arr[0]);
        arr[1] = Number(arr[1]);

        return arr;
      })
      console.log(points)
      axios.post(`http://localhost:1337/api/plots`, {
        data: {
          level: this.newPlot.level,
          points
        }
      }, {
        headers: {
          Authorization: `Bearer 3c6fe90c9d56419452691a3f05b6a818e300b953508755f21e7ed99711668d7c525a4300a7354f5c8646c96b7a8349a9975cce6392d0e7b3ec758ea64729bd012a8cc320516ffd23f33d118cc72e73af5712f7394653da3ba7d4e568c8c857f4d1cc6d87ab9155181a96b4c370abf68dd31ddf239e27cdeaed16d48c8dbf0662`
        }
      }).then().catch()

      this.close()
      this.$emit('setplots', await this.getPlots())
      this.$emit('createmap')
    }
  },
};
</script>

<style>
.modal-backdrop {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background: #FFFFFF;
  box-shadow: 2px 2px 20px 1px;
  overflow-x: auto;
  display: flex;
  flex-direction: column;
}

.modal-header,
.modal-footer {
  padding: 15px;
  display: flex;
}

.modal-header {
  position: relative;
  border-bottom: 1px solid #eeeeee;
  color: #4AAE9B;
  justify-content: space-between;
}

.modal-footer {
  border-top: 1px solid #eeeeee;
  flex-direction: column;
  justify-content: flex-end;
}

.modal-body {
  position: relative;
  padding: 20px 10px;
}

.btn-close {
  position: absolute;
  top: 0;
  right: 0;
  border: none;
  font-size: 20px;
  padding: 10px;
  cursor: pointer;
  font-weight: bold;
  color: #4AAE9B;
  background: transparent;
}

.btn-green {
  color: white;
  background: #4AAE9B;
  border: 1px solid #4AAE9B;
  border-radius: 2px;
}
</style>
