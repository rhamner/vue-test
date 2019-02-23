<template>
  <div id="app">
    <v-container grid-list-md>
      <v-layout row wrap>
        <v-flex md6>
          <v-card class='summary' color="blue-grey lighten-5">    
            This is a simple example of using <a href="https://plot.ly/javascript/">plotly</a> in a <a href="https://vuejs.org/">Vue</a> app built with <a href="https://vuetifyjs.com/en/">Vuetify</a>. These little boxes 
            are cards. They are responsive because I'm using the flex/grid system in Vuetify. The plot
            is a separate component.<br><br>
            The custom equation is evaluated using <a href="https://www.npmjs.com/package/bigeval">BigEval</a>. You can try things like <i>y = 4*pow(x, 2)</i> and <i>y = 
            A*B*B + x</i>. It accepts javascript syntax for the most part, but don't add Math. in front of the functions.
            <br><br>
            Change the settings on the right to update the plot.
          </v-card>
        </v-flex>
        <v-flex md6>
          <v-card class="selectors" color="blue-grey lighten-5">
            <v-layout>
            <v-flex md6>
            <v-text-field
                type="number"
                label="A"
                step=0.1
                v-model="A">
              </v-text-field>
              </v-flex>
              <v-flex md6>
              <v-text-field
                type="number"
                label="B"
                step=0.1
                maxlength=3
                v-model="B">
              </v-text-field>
              </v-flex>
            </v-layout> 
            <v-layout>
              <v-flex md6>
            <v-radio-group v-model="selectedFunction" column>
              <v-radio
                v-for="(label, index) of radioLabels"
                :key=index
                :label=label
                :value=index
              ></v-radio>
            </v-radio-group>
              </v-flex>                  
              <v-flex md6>
                <v-text-field
                  v-show="selectedFunction===3"
                  label="type equation"
                  @change="updatePlot"
                  v-model="equation"
                  :rules="[validEquation]">
                </v-text-field>
              </v-flex>
            </v-layout>
          </v-card>
        </v-flex>
      </v-layout>
      <v-flex style="margin-top: 5px">
        <v-card color="blue-grey lighten-5">
          <plotlygraph :propData="plotInfo" divId="plot" @afterplot="plotChanged" style="height: 300px"></plotlygraph>
          <div>{{status}}</div>
        </v-card>
      </v-flex>
    </v-container>
  </div>
</template>
<script>

import plotlygraph from './components/PlotlyGraph.vue'
var BigEval = require('bigeval');

export default {
  name: 'app',
  components: {
    plotlygraph
  },
  data: function() {
    return ({
      radioLabels: ['y = A*sin(B*x)', 'y = A*cos(B*x)', 'y = A*(B^x)', 'custom'],
      A: 1,
      B: 1,
      selectedFunction: 1,
      plotInfo: { 
        data: [{
          x: [0, 1, 2, 3],
          y: [4, 1, 2, 1],
          type: 'scatter'
        }]
      },
      equation: "A + B*x",
      eq: new BigEval(),
      status: ''
    })
  },
  mounted() {
    this.updatePlot();
    this.$watch('selectedFunction', this.updatePlot);
    this.$watch('A', this.updatePlot);
    this.$watch('B', this.updatePlot);
  },
  methods: {
    updatePlot() {
      let x = [];
      let y = [];
      for (let i = -5; i <= 5; i += 0.01) {
        x.push(i);
        if (this.selectedFunction === 0) {
          y.push(this.A*Math.sin(this.B*i));
        } else if (this.selectedFunction === 1) {
          y.push(this.A*Math.cos(this.B*i));
        } else if (this.selectedFunction === 2) {
          y.push(this.A*Math.pow(this.B, i));
        } else {

          // BigEval takes a string representing an equation, and evaluates it. It returns an ERROR if the syntax is invalid
          let cmd = this.equation.replace(/y/g, '').replace(/ /g, '').replace(/=/g, '').replace(/A/g, this.A.toString()).replace(/B/g, this.B.toString()).replace(/x/g, i.toString());
          let res = this.eq.exec(cmd);
          y.push(res);
        }
      }
      let data = [{
        x: x,
        y: y,
        type: 'scatter',
        line: {
          width: 4
        }
      }];
      let layout = {

        // get rid of all the extra whitespace around plots
        margin: {
          t: 25,
          b: 25,
          r: 25,
          l: 25
        },
        paper_bgcolor:"#ECEFF1"
      }
      let config = {

        // newish in plotly...auto-resizes on window resize
        responsive: true
      }
      this.plotInfo = { data: data, layout: layout, config: config }
    },
    validEquation() {

      // make sure they have an equation at all
      if (this.equation.length === 0) {
        return "You have to enter something"
      }

      // not robust, but just check x = -1...if error of NaN, say it's invalid
      let cmd = this.equation.replace(/A/g, this.A.toString()).replace(/B/g, this.B.toString()).replace(/x/g, '-1');
      let res = this.eq.exec(cmd);
      if (res === "ERROR" || isNaN(res)) {
        return this.equation + " is invalid"
      } else {
        return true;
      }
    },
    plotChanged() {
      this.status = 'Plot last updated at ' + new Date();
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

.summary {
  font-size: 16px;
  padding: 15px;
  text-align: left;
  min-height: 280px;
}

.selectors {
  min-height: 280px;
  padding: 15px;
  padding-left: 30px;
}
</style>
