<template>
  <div class="krt-dc-stacked-bar" :id="id">
    <a class="reset" style="display: none">reset</a>
  </div>
</template>

<script lang='js'>

import d3 from "d3"
import dc from 'dc'
import Base from './_base'
import Store from '../store'

function _generateReducer(idx=0) {
  return function() {
    const dim = Store.getDimension(this.dimensionName);
    const _reducer = this.getReducerExtractor;
    dim.group().reduceSum((d) => {
      console.log(_reducer(d)[idx])
    })
    return dim.group().reduceSum((d) => _reducer(d)[idx]);
  }
}

export default {
  extends: Base,

  props: {
    chartType: {
      type: String,
      default: 'barChart'
    },
    labels: {
      type: Array
    },
    xAxisLabel: {
      type: String,
      default: 'xAxisLabel'
    },
    yAxisLabel: {
      type: String,
      default: 'yAxisLabel'
    },
    renderLabel: {
      type: Boolean,
      default: true
    },
    renderHorizontalGridLines: {
      type: Boolean,
      default: true
    },
    elasticY: {
      type: Boolean,
      default: true
    },
    legendX: {
      type: Number,
      default: 0
    },
    legendY: {
      type: Number,
      default: 0
    },
    // brushOn: {
    //   type: Boolean,
    //   default: false
    // },
    // clipPadding: {
    //   type: Number,
    //   default: 10
    // }
  },
  computed: {
    reducer: _generateReducer(0)
  },
  mounted: function() {
    const chart = this.chart;
    const barNum = this.labels.length

    chart
      .group(this.reducer, this.labels[0])
      .x(d3.scale.ordinal())
      .xUnits(dc.units.ordinal)
      .brushOn(false)
      .clipPadding(10)
      .elasticY(this.elasticY)
      .xAxisLabel(this.xAxisLabel)
      .yAxisLabel(this.yAxisLabel)
      .renderLabel(this.renderLabel)
      .legend(dc.legend().x(this.legendX).y(this.legendY))
      .renderHorizontalGridLines(this.renderHorizontalGridLines)
    // stack
    for (let i=1; i<barNum; i++) {
      chart.stack(_generateReducer(i).apply(this), this.labels[i]);
    }
    // reverse dc.legend() order
    // See: http://stackoverflow.com/questions/39811210/dc-charts-change-legend-order
    dc.override(chart, 'legendables', function() {
        var items = chart._legendables();
        return items.reverse();
    });

    return chart.render();
  }
}
</script>