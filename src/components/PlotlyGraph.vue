<template>
  <div
    class="plotlygraph"
    ref="container"
    :id='divId'
    :style="{
      height: (height) +  '%',
      width: (width) + '%'
    }"/>
</template>

<script>
import Plotly from 'plotly.js/dist/plotly';

export default {
  name: 'plotlygraph',
  props: {
    propData: {
      type: Object,
      required: true
    },
    divId: {
      type: String,
      required: true
    },
    height: {
      type: Number,
      default: 100
    },
    width: {
      type: Number,
      default: 95
    },
  },
  data: function() {
    return {
      doubleClickTime: 0,
      doubleClickThreshold: 400
    };
  },
  mounted() {
    this.newPlot();
    this.bindEvents();
  },
  created() {
    this.$watch('propData', this.update); // was previously setting deep = true...turned off to block infinite loops but not sure why it was there
    this.$watch('editable', this.update);
  },
  beforeDestroy() {
    this.unbindEvents();
  },
  computed: {
    plotDiv() {
      return this.$refs.container;
    }
  },
  methods: {
    bindEvents() {
      this.plotDiv.on('plotly_hover', this.hover);
      this.plotDiv.on('plotly_unhover', this.unhover);
      this.plotDiv.on('plotly_relayout', this.relayout);
      this.plotDiv.on('plotly_restyle', this.restyle);
      this.plotDiv.on('plotly_click', this.click);
      this.plotDiv.on('plotly_selected', this.selected);
      this.plotDiv.on('plotly_selecting', this.selecting);
      this.plotDiv.on('plotly_autosize', this.autosize);
      this.plotDiv.on('plotly_deselect', this.deselect);
      this.plotDiv.on('plotly_doubleclick', this.doubleclick);
      this.plotDiv.on('plotly_redraw', this.redraw);
      this.plotDiv.on('plotly_animated', this.animated);
      this.plotDiv.on('plotly_afterplot', this.afterplot);
    },
    unbindEvents() {
      if (!this.plotDiv['off']) return;
      this.plotDiv.off('plotly_hover', this.hover);
      this.plotDiv.off('plotly_unhover', this.unhover);
      this.plotDiv.off('plotly_relayout', this.relayout);
      this.plotDiv.off('plotly_restyle', this.restyle);
      this.plotDiv.off('plotly_click', this.click);
      this.plotDiv.off('plotly_selected', this.selected);
      this.plotDiv.off('plotly_selecting', this.selecting);
      this.plotDiv.off('plotly_autosize', this.autosize);
      this.plotDiv.off('plotly_deselect', this.deselect);
      this.plotDiv.off('plotly_doubleclick', this.doubleclick);
      this.plotDiv.off('plotly_redraw', this.redraw);
      this.plotDiv.off('plotly_animated', this.animated);
      this.plotDiv.off('plotly_afterplot', this.afterplot);
    },
    // publishes divId and points hovered over when hover starts
    hover(eventData) {
      this.$emit('hover', this.divId, eventData);
    },

    // publishes divId and points that were previously hovered over when hover stops
    unhover(eventData) {
      this.$emit('unhover', this.divId, eventData.points);
    },

    // Publishes divId, new axes, new plot pixel dimensions,
    // and full event info containing everything anytime the plot redraws
    relayout(eventData) {
      var axes = {
        xaxis: this.plotDiv.layout.xaxis,
        yaxis: this.plotDiv.layout.yaxis
      };
      var dimensions = {
        width: this.plotDiv.clientWidth,
        height: this.plotDiv.clientHeight
      };
      this.$emit('relayout', this.divId, axes, dimensions, eventData);
    },

    restyle(eventData) {

    },

    // Publishes divId and points clicked when a point is clicked
    click(eventData) {
      // Click fires once on single and twice on double clicks
      // We only care about single clicks.
      //This checks to give the doubleclick event 500 ms to fire, and does nothing if so
      var t0 = Date.now();
      var _this = this;
      if ((t0 - this.doubleClickTime) > this.doubleClickThreshold) {
        setTimeout(function() {
          if ((t0 - _this.doubleClickTime) > _this.doubleClickThreshold) {
            _this.singleClickHandler(eventData);
          }
        }, this.doubleClickThreshold);
      }
    },

    singleClickHandler(eventData) {
      this.$emit('click', this.divId, eventData.points);
    },

    // Publishes divId, and an object containing lasso points
    // and points contained in the lasso after lasso selection
    selected(eventData) {
      if (!eventData) return;

      // clear selected if you selected nothing
      if (eventData.points.length === 0) {
        Plotly.restyle(this.divId, {selectedpoints: [null]});
        return;
      }
      this.$emit('selected', this.divId, eventData);
    },

    // Triggered when lasso selection is happening
    selecting(eventData) {
    },

    // Triggered whenever size of plot changes
    autosize(eventData) {
    },

    // Triggered when you double-click to turn off the lasso or box selection
    deselect(eventData) {
      this.$emit('deselect', this.divId);
    },

    // Double click in plotly is a plot level event,
    // so all that's published out is the plot's divId
    doubleclick() {
      this.doubleClickTime = Date.now();
      this.$emit('doubleclick', this.divId);
    },

    // Redraw in plotly is a plot level event,
    // so all that's published out is the plot's divId
    redraw() {
      this.$emit('redraw', this.divId);
    },

    // Triggered when animated
    animated() {
    },

    afterplot() {
      this.$emit('afterplot');
    },

    addTrace(newTrace) {
      Plotly.addTraces(this.divId, newTrace);
    },
    deleteTrace() {
      Plotly.deleteTraces(this.divId, [-1]);
    },
    newPlot() {
      Plotly.newPlot(this.divId, this.propData.data, this.propData.layout, this.propData.config);
    },
    update() {
      Plotly.react(this.divId, this.propData.data, this.propData.layout, this.propData.config);
    }
  }
};
</script>

<style>
.plotlygraph {
  margin-left: auto;
  margin-right: auto;
}
</style>

