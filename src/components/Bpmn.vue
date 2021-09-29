<template>
  <div>
    <div class="content with-diagram" id="container">
      <div class="canvas" id="canvas"></div>
      <div class="properties-panel-parent" id="properties"></div>
    </div>
    <button id="save-button" v-on:click="save()">save</button>
  </div>
</template>

<script scoped>
import BpmnJS from 'bpmn-js/lib/Modeler';
import propertiesPanelModule from 'bpmn-js-properties-panel';
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/bpmn';
import axios from 'axios';

const diagramUrl = 'https://cdn.staticaly.com/gh/bpmn-io/bpmn-js-examples/dfceecba/starter/diagram.bpmn';
// const saveUrl = 'http://localhost:8080/rest/process/save';
const saveUrl = '/rest/process/save';

export default {
  name: 'bpmn',
  data () {
    return {
      msg: 'bpmn component'
    }
  },
  mounted() {
    this.bjs = new BpmnJS({
      container: '#canvas',
      propertiesPanel: {
        parent: '#properties'
      },
      additionalModules: [
        propertiesPanelModule,
        propertiesProviderModule
      ]
    });
    axios.get(diagramUrl)
    .then((resp) => {
      this.loadDiagram(resp.data);
    })
    .catch((err) => {
      console.error(err);
    });
  },
  methods: {
    async loadDiagram(bpmnXML) {
      try {
        await this.bjs.importXML(bpmnXML);
        let canvas = this.bjs.get('canvas');
        canvas.zoom('fit-viewport');
      } catch (err) {
        console.error('could not import BPMN 2.0 diagram', err);
      }
    },
    async exportDiagram() {
      try {
        let result = await this.bjs.saveXML({ format: true });
        console.log('DIAGRAM', result);
        return result.xml;
      } catch (err) {
        console.error('could not save BPMN 2.0 diagram', err);
      }
    },
    async exportImage() {
      try {
        let result = await this.bjs.saveSVG();
        console.log('SVG', result);
        return result.svg;
      } catch (err) {
        console.error('could not save BPMN 2.0 diagram', err);
      }
    },
    async save() {
      let xml = await this.exportDiagram();
      let svg = await this.exportImage();
      let params = new URLSearchParams();
      params.append('bpmn', xml);
      params.append('svg', svg);
      axios.post(saveUrl, params)
      .then((resp) => {
        alert(resp.data);
      })
      .catch((err) => {
        console.error(err);
      });
    }
  }
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
<style scoped src="bpmn-js/dist/assets/diagram-js.css"></style>
<style scoped src="bpmn-js/dist/assets/bpmn-font/css/bpmn-embedded.css"></style>
<style scoped src="bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css"></style>
<style scoped src="../assets/css/app.css"></style>
