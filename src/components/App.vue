<template>
  <div id="app">
    <select v-model="selectedId">
      <option v-for="port in ports" :key="port.id" :value="port.id">{{port.name}}</option>
    </select>
    <div>Chord: {{chord}}</div>
    <div>{{notes}}</div>
    
    <!--
        <svg width=300 height=50>
      <rect v-for="k in keys" :key="k"></rect>
    </svg>
    -->

  </div>
</template>

<script>
var synth;
var MidiIn
export default {
  name: "app",
  data() {
    return {
      ports: [],
      selectedId: "",
      chord: "",
      notes: []
    };
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

    MidiIn = (e)=>{
      var d = e.data
      if(d[0] === 144){
        this.notes.push(d[1])
      }
      if(d[0] === 128){
        this.notes = this.notes.filter(n => n !== d[1])
      }
      synth.send(e.data)
    }
    
  }
};
</script>
