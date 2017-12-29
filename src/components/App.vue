<template>
  <div id="app">
    <div>
    <select v-model="selectedId">
      <option v-for="port in ports" :key="port.id" :value="port.id">{{port.name}}</option>
    </select>

    </div>

    <svg width=300 height=300>
     <polygon :points="points"></polygon>
     <text alignment-baseline="middle" text-anchor="middle" v-for="(label, idx) in cofLabels" :key="idx" :x="label.x" :y="label.y">{{label.name}}</text>
    </svg>

    <div>Chord: {{chords}}</div>
    <div>{{notenames}}</div>
    <div>{{cof}}</div>
    <!--
        <svg width=300 height=50>
      <rect v-for="k in keys" :key="k"></rect>
    </svg>
    -->

  </div>
</template>

<script>
var piu = require("piu");
var _ = require("lodash");
var teoria = require("teoria");
var synth;
var MidiIn;
var dict = [
  { cof: 0, pos: 0, name: "C" },
  { cof: 7, pos: 0.5, name: "Db" },
  { cof: 2, pos: 1, name: "D" },
  { cof: 9, pos: 1.5, name: "Eb" },
  { cof: 4, pos: 2, name: "E" },
  { cof: 11, pos: 3, name: "F" },
  { cof: 6, pos: 3.5, name: "Gb" },
  { cof: 1, pos: 4, name: "G" },
  { cof: 8, pos: 4.5, name: "Ab" },
  { cof: 3, pos: 5, name: "A" },
  { cof: 10, pos: 5.5, name: "Bb" },
  { cof: 5, pos: 6, name: "B" }
];
function valueToPoint(value, index, total) {
  var x = 0;
  var y = -value * 0.8;
  var angle = Math.PI * 2 / total * index;
  var cos = Math.cos(angle);
  var sin = Math.sin(angle);
  var tx = x * cos - y * sin + 150;
  var ty = x * sin + y * cos + 150;
  return {
    x: tx,
    y: ty
  };
}

export default {
  name: "app",
  data() {
    return {
      ports: [],
      selectedId: "",
      notes: [],
      stats: [0, 4, 5],
    };
  },
  computed: {
    points() {
      return this.cof
        .map((stat, i) => {
          var point = valueToPoint(100, stat, 12);
          return point.x + "," + point.y;
        })
        .join(" ");
    },
    notenames() {
      return this.notes.map(i => {
        return dict[i % 12].name;
      });
    },
    cof() {
      if (this.notes.length === 0) {
        return [];
      }
      return _.uniq(this.notes.map(i => dict[i % 12].cof)).slice().sort(
        (a, b) => a - b
      );
    },
    chords() {
      return piu
        .infer(this.notenames.map(teoria.note))
        .map(piu.name)
        .map(n => n.replace(/undefined/, "?"));
    },
    cofLabels() {
      return dict.slice().sort((a,b)=>(a.cof - b.cof)).map((a, i)=>{
        const p = valueToPoint(120, i, 12);
        return {
          name: a.name,
          x: p.x,
          y: p.y
        }
      })
    },

  },
  watch: {
    selectedId(val, old) {
      const oldPort = this.ports.find(i => i.id === old);
      if (oldPort) {
        oldPort.removeEventListener("midimessage", MidiIn);
      }
      const newPort = this.ports.find(i => i.id === val);
      if (newPort) {
        newPort.addEventListener("midimessage", MidiIn);
      }
    }
  },
  mounted() {
    synth = new WebAudioTinySynth({ quality: 0, useReverb: 0, voices: 32 });
    navigator.requestMIDIAccess().then(midiAccess => {
      for (let input of midiAccess.inputs.values()) {
        this.ports.push(input);
      }
      this.selectedId = this.ports[0].id;
    });

    MidiIn = e => {
      var d = e.data;
      if (d[0] === 144) {
        this.notes.push(d[1]);
      }
      if (d[0] === 128) {
        this.notes = this.notes.filter(n => n !== d[1]);
      }
      synth.send(e.data);
    };
  }
};
</script>
<style>
body {
  background: #eee;
}
svg {
  background: white;
}
polygon {
  fill: #42b983;
  opacity: 0.75;
}
</style>
