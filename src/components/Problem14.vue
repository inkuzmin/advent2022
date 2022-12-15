<template>
  <article>
    <h2>--- Day 14: Regolith Reservoir ---</h2>

    <p>The distress signal leads you to a giant waterfall! Actually, hang on - the signal seems like it's coming from the waterfall itself, and that doesn't make any sense. However, you do notice a little path that leads <em>behind</em> the waterfall.</p>
    <p><a href="https://adventofcode.com/2022/day/14">&#60;…&#62;</a></p>
    <p>Using your scan, simulate the falling sand. <em>How many units of sand come to rest before sand starts flowing into the abyss below?</em></p>

    upload input: <input type="file" @change="upload"> <br>
    <input type="checkbox" v-model="animate">animate <br>
    <input type="checkbox" v-model="floor">floor <br>
    <template v-if="!animate">
    skip by: <input type="number" v-model="speed">
    </template>

    <p>Number of grains: <em>{{ n }}</em></p>

    <div ref="simul" class="simul">

    </div>
  </article>
</template>

<script>
/* eslint-disable */

import input from '../inputs/day14.txt';

let rock = null;
let sand = null;

window.sand = sand;
window.rock = rock;

function initMx(mx) {
  for (let i = 0; i < 1000; i++) {
    let row = [];
    for (let j = 0; j < 1000; j++) {
      row.push(0);
    }
    mx.push(row)
  }
  return mx;
}

let iteration = 0;
let maxDepth = 0;

function fillRock(a, b) {
  for (let i = a[0]; i <= b[0]; i++) {
    rock[a[1]][i] = 1;

    if (a[1] > maxDepth) {
      maxDepth = a[1];
    }

  }
  for (let i = a[1]; i <= b[1]; i++) {
    rock[i][a[0]] = 1;

  }
}

export default {
  name: "Problem14",
  data() {
    return {
      currentSand: [0, 500],
      animate: true,
      floor: false,
      speed: 10,
      n: 0,
      input: input,
    }
  },
  watch: {
    input() {
      this.init();
    },
    animate() {
      this.render();
    },
    speed() {
      this.render();
    },
    floor(newVal) {
      if (newVal) {
        for (let i = 0; i < 1000; i++) {
          rock[maxDepth + 2][i] = 1;
        }
      } else {
        for (let i = 0; i < 1000; i++) {
          rock[maxDepth + 2][i] = 0;
        }
      }
    }
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {

      rock = initMx([]);
      sand = initMx([]);

      maxDepth = 0;
      iteration = -1;

      let lines = this.input.split('\n');
      for (let line of lines) {
        let specs = line.split(' -> ');
        for (let i = 0; i < specs.length; i++) {
          let point = specs[i].split(',').map(v => parseInt(v));
          let nextPoint = specs[i+1]?.split(',').map(v => parseInt(v));

          if (nextPoint) {
            if (point[0] < nextPoint[0] || point[1] < nextPoint[1]) {
              fillRock(point, nextPoint);
            } else {
              fillRock(nextPoint, point);
            }
          }
        }
      }

      this.render();

      this.moveSand();
    },

    upload(e) {
      let fileData = e.target.files[0]
      let reader = new FileReader();
      reader.readAsText(fileData);
      reader.addEventListener("load",(event) => {
        this.input = event.target.result;
      })
    },

    doesCollide(x, y) {
        return rock[x][y] || sand[x][y];
    },

    nextSand(sand) {
      let x = sand[0];
      let y = sand[1];

      if (!this.doesCollide(x+1, y)) {
        return [x+1, y];
      } else if (!this.doesCollide(x+1, y-1)) {
        return [x+1, y-1];
      } else if (!this.doesCollide(x+1, y+1)) {
        return [x+1, y+1];
      } else {
        return [x, y];
      }
    },

    moveSand(currentSand = [0, 500]) {
        if (iteration === -1) {
          return;
        }

        let nextSand = this.nextSand(currentSand);

        if ( (currentSand[0] != nextSand[0]) || (currentSand[1] != nextSand[1]) ) {

          if (nextSand[0] >= maxDepth + 5) {
            this.render();
            setTimeout(()=>{this.moveSand()}, 1000);
          } else {
            this.currentSand = nextSand;

            if (this.animate) {
              this.render();
              setTimeout(() => {
                this.moveSand(nextSand);
              }, 100);
            } else {
                this.moveSand(nextSand);
            }
          }
        } else if ( (currentSand[0] == 0) && (currentSand[1] == 500) ) {
        } else {
          sand[currentSand[0]][currentSand[1]] = 1;



          if (iteration % this.speed == 0) {
            iteration = 0;
            this.render();
          }
          setTimeout(()=>{this.moveSand()}, 0);

          iteration += 1;
        }

    },
    render() {
      let el = this.$refs['simul'];

      let innerText = ''

      for (let i = 0; i < maxDepth + 5; i++) {
        for (let j = 0; j < 1000; j++) {
          if (i === 0 && j === 500) {
            innerText += '<span class="source">+</span>'
          } else if (i === this.currentSand[0] && j === this.currentSand[1]) {
            innerText += '<span class="sand">o</span>'
          } else {
            if (rock[i][j]) {
              innerText += '<span class="rock">#</span>'
            } else if (sand[i][j]) {
              innerText += '<span class="sand">o</span>'
            } else {
              innerText += '.'
            }
          }
        }
        innerText += '\n'
      }

      let t = 0;
      for (let i of sand) {
        for (let j of i) {
          if (j) {
            t += 1;
          }
        }
      }
      this.n = t;

      el.innerHTML = innerText;

    }
  }
}
</script>

<style>
  .rock {
    color: grey;
  }
  .sand {
    color: yellow;
  }
  .source {
    color: white;
  }
</style>

<style scoped>

  .simul {
    font-size: 5px;
    position: absolute;
    left: -50%;
    color: #333340;
  }

</style>