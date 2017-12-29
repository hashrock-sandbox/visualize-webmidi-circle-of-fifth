<template>
  <div id="app">
    <div>
    <select v-model="selectedId">
      <option v-for="port in ports" :key="port.id" :value="port.id">{{port.name}}</option>
    </select>

    </div>

    <svg width=300 height=300>
    </svg>

    <div>Chord: {{chords}}</div>
    <div>{{notenames}}</div>
    
    <!--
        <svg width=300 height=50>
      <rect v-for="k in keys" :key="k"></rect>
    </svg>
    -->

  </div>
</template>

<script>
var piu = require("piu");
var teoria = require("teoria");
var synth;
var MidiIn;
var dict = [
  { pos: 0, name: "C" },
  { pos: 0.5, name: "Db" },
  { pos: 1, name: "D" },
  { pos: 1.5, name: "Eb" },
  { pos: 2, name: "E" },
  { pos: 3, name: "F" },
  { pos: 3.5, name: "Gb" },
  { pos: 4, name: "G" },
  { pos: 4.5, name: "Ab" },
  { pos: 5, name: "A" },
  { pos: 5.5, name: "Bb" },
  { pos: 6, name: "B" }
];

export default {
  name: "app",
  data() {
    return {
      ports: [],
      selectedId: "",
      notes: []
    };
  },
  computed: {
    notenames() {
      return this.notes.map(i => {
        return dict[i % 12].name;
      });
    },
    chords() {
      return piu
        .infer(this.notenames.map(teoria.note))
        .map(piu.name)
        .map(n => n.replace(/undefined/, "?"))
    }
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
body{
  background: #EEE;
}
svg{
  background: white;
}
</style>
