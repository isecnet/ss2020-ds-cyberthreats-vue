<template>
  <vue-p5 class="p5" @setup="setup" @draw="draw" @mouseclicked="mouseclicked"></vue-p5>
</template>

<script>
// import axios from 'axios'
import VueP5 from 'vue-p5';

export default {
  components: {
    "vue-p5": VueP5
  },
  data: function () {
    return {
      attack: Object,
      zoom: 4,
      yOffset: 100,
      attacks: {},
      maxCircleRadius: 100,
    }
  },
  mounted() {
    this.sockets.subscribe('attack', (data) => {
      this.attack = data;
      const country = data.Destination;
      if (this.attacks[country] !== undefined) {
        this.attacks[country].push(data);
      } else {
        this.attacks[country] = [];
        this.maxCircleRadius += 20;
        this.attacks[country].push(data);
      }
    });
  },
  methods: {
    setup(sketch) {
      //sketch.createCanvas(1800, 1000);
      sketch.createCanvas(sketch.displayWidth, sketch.displayHeight);
    },
    draw(sketch) {
      const hue = Math.floor(sketch.map(sketch.mouseX, 0, sketch.displayWidth, 0, 360));
      const light = Math.floor(sketch.map(sketch.mouseY, 0, sketch.displayHeight, 100, 10));
      const color = sketch.color(`hsl(${hue}, 100%, ${light}%)`);
      sketch.clear();
      sketch.background(0);
      sketch.circle(sketch.mouseX, sketch.mouseY, 10);
      sketch.translate(sketch.width / 2, sketch.height / 2);
      sketch.scale(1, 1);
      sketch.noStroke(); // Don't draw a stroke around shapes
      sketch.fill(color); // Use color variable 'c' as fill color
      
      let attackCount = 0;

      for (const key in this.attacks) {
        const country = this.attacks[key];
        attackCount += country.length;
      }

      for (const key in this.attacks) {
        const country = this.attacks[key];
        const radius = (this.maxCircleRadius / attackCount) * country.length;
        sketch.circle(country[0].DestinationCoords[1] * this.zoom, -country[0].DestinationCoords[0] * this.zoom * 2 + this.yOffset, radius);
        //sketch.fill(0);
        //sketch.text(country[0].Destination, country[0].DestinationCoords[1] * this.zoom, -country[0].DestinationCoords[0] * this.zoom * 2 + this.yOffset);
      }
    },
    mouseclicked() {
      this.$emit('click-event');
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.p5 {
  cursor: none;
}
</style>
