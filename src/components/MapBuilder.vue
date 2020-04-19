<template>
  <div class="map-container">
  <div class="map-list">
    <div class="map-actions">
      <div class="map-export-btn" @click="exportJson">Export to JSON</div>
      <div class="map-export-name">
        <input v-model="levelName">
      </div>
<!--      <div class="map-export-info">-->
<!--        <div><strong>Max blocks:</strong>{{imgs.length}}/{{blockLimit}}</div>-->
<!--        <div><strong>Need station:</strong>{{stations.length}}/{{needStations}}</div>-->
<!--        <div><strong>Need reactors:</strong>{{reactors.length}}/{{needReactors}}</div>-->
<!--      </div>-->
    </div>
    <div @click="selectTemplate(ind)"
    :key="'key' + ind" class="map-list-item" v-for="(temp, ind) in templates"
    :class='{"map-list-item--selected": ind == selectedTemplate}'>
      <img
      :src="temp.image" style="width: 100%"/>
    </div>
  </div>
  <div class="map-builder">
    <div class="templCover" v-if="templCover"
         :style='{
      "top": templCover.pos.x + "px", "left": templCover.pos.y + "px",
      "width": (templCover.size.w) + "px", "height": (templCover.size.h) + "px",
      "z-index": (reactorMode)?9:"auto",
      "background-image": "url(" + templCover.image + ")"
    }'></div>

    <template v-for="block in blocks">
      <div @click="selectBlock(block)"
           @mouseover="mouseOver(block)"
      :key="'key' + block.pos.i + 'x' + block.pos.j"
      :class='{"gridBlock--selected": (selectedBlock && block.id == selectedBlock.id)}'
      class="gridBlock"
      :style='{"top": block.pos.x + "px", "left": block.pos.y + "px", "z-index": (reactorMode)?9:"auto"}'></div>
    </template>
      <template v-for="(img, ind) in imgs">
        <img
        @click="removeImg(ind)"
        :src="img.image"
        :key="'key' + ind"
        class="blockImage"
        :style='{"top": img.pos.x + "px", "left": img.pos.y + "px", "z-index": img.isback?"0":"1"}'/>
      </template>
  </div>
    <div class="result-modal-back" v-if="modalResultShow">
      <div class="result-modal">
        <div class="result-modal-head">
          Export result:
        </div>
        <div class="result-modal-body">
          <textarea style="width: 100%;" v-model="jsonResult" rows="15"></textarea>
        </div>
        <div class="result-modal-footer">
          <button @click="modalResultShow = false">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'MapBuilder',
  props: {
    msg: String
  },
  data() {
  return {
    reactorMode: false,
    levelName: null,
    reactors: [],
    stations: [],
    blockLimit:25,
    blockMin:3,
    needReactors:4,
    needStations:1,
  playerCount: 0,
  exitCount: 0,
  width: 100,
  height: 100,
  blocks: [],
  templates: [],
  imgs: [],
    modalResultShow: false,
    jsonResult: null,
  selectedBlock: null,
  selectedTemplate: null,
    templCover: {
    image: null,
    pos: {
      x: 0, y:0
    },
      size: {w:32,h:32}
    }
  }
  },
  mounted() {
    this.generateBlocks();
    this.loadTemplates();
  },
  methods: {
    exportJson() {
      if(this.playerCount!==1) {
        alert('need 1 robot in map');
        return;
      }
      if(this.exitCount<1) {
        alert('need minimum 1 exit in map');
        return;
      }
      // if(this.stations && this.stations.length < this.needStations) {
      //   alert('Add ' + this.needStations + ' stations to the map');
      //   return;
      // }
      // if(this.reactors && this.reactors.length < this.needReactors) {
      //   alert('Add ' + this.needReactors + ' reactors to the map');
      //   return;
      // }
      // if(this.imgs.length < this.blockMin) {
      //   alert('Add minimum ' + this.blockMin + ' blocks to the map');
      //   return;
      // }
      let finalJson = [];
      for(let i=0;i<this.imgs.length;i++) {
        let im = this.imgs[i];
        finalJson.push({
          "type": im.type,
          "pos": {
            "j": this.div(im.pos.x, 32),
            "i": this.div(im.pos.y, 32)
          }
        })
      }

      this.jsonResult = JSON.stringify({
        title: 'level_1',
        map: finalJson
      });
      this.modalResultShow = true;
    },
    selectTemplate(ind) {
      let templ = this.templates[ind];
      if(templ.isback) {
        this.reactorMode = true;
      } else {
        this.reactorMode = false;
      }
      this.selectedTemplate = ind;
    },
    div(val, by){
      return (val - val % by) / by;
    },
    updateCoordinates(event) {
      if(this.selectedTemplate) {
      this.selectedTemplate.pos.x = this.div(event.clientX/32);
      this.selectedTemplate.pos.y = this.div(event.clientY/32);
      }
    },
    loadTemplates() {
      let list = [
      {"image": "/blocks/platform_icy_croped.png","width": 32,"height":16,"type":"platform_icy", "isback": false},
      {"image": "/blocks/robot_caring.png","width": 32,"height":32,"type":"robot_caring", "isback": false},
      {"image": "/blocks/armature1.png","width": 32,"height":32,"type":"armature1", "isback": true},
      {"image": "/blocks/armature2.png","width": 32,"height":32,"type":"armature2", "isback": true},
      {"image": "/blocks/kafel_big.png","width": 32,"height":32,"type":"kafel", "isback": true},
      {"image": "/blocks/racks.png","width": 32,"height":32,"type":"racks", "isback": true},
      {"image": "/blocks/alphalth_snowy_p.png","width": 32,"height":32,"type":"alphalth_snowy_p", "isback": false},
      {"image": "/blocks/asphalth.png","width": 32,"height":32,"type":"asphalth", "isback": false},
      {"image": "/blocks/snow.png","width": 32,"height":32,"type":"snow", "isback": true},
      {"image": "/blocks/exit.png","width": 32,"height":32,"type":"exit", "isback": false},
      // {"image": "/blocks/block_green_02.png","width": 160,"height":224,"type":"green_009_7x7_1x0x1x0"},
      // {"image": "/blocks/block_green_03.png","width": 192,"height":256,"type":"green_010_6x8_0x0x1x0"},
      // {"image": "/blocks/block_orange_02.png","width": 160,"height":160,"type":"orange_002_5x5_1x1x1x1"},
      // {"image": "/blocks/block_orange_03.png","width": 160,"height":224,"type":"orange_003_5x7_1x1x0x0"},
      // {"image": "/blocks/block_orange_04.png","width": 160,"height":224,"type":"orange_004_5x7_1x1x1x0"},
      // {"image": "/blocks/block_orange_05.png","width": 160,"height":224,"type":"orange_007_5x8_1x1x1x0"},
      // {"image": "/blocks/block_orange_06.png","width": 224,"height":224,"type":"orange_012_7x7_0x1x1x1"},
      // {"image": "/blocks/block_orange_s_01.png","width": 224,"height":224,"type":"orange_001_7x7_1x1x1x1"},
      // {"image": "/blocks/block_red_big_01.png","width": 160,"height":224,"type":"red_006_5x7_1x1x0x0"},
      // {"image": "/blocks/block_red_big_02.png","width": 160,"height":256,"type":"red_005_5x8_1x0x1x1"},
      // {"image": "/blocks/block_red_big_03.png","width": 160,"height":224,"type":"red_011_5x7_0x0x0x1"},
      // {"image": "/blocks/space_station.png","width": 480,"height":192,"type":"station"},
      // {"image": "/blocks/reactorBlock.png","width": 32,"height":32,"type":"reactor"}
      ];
      this.templates = list;
      this.selectedTemplate = 0;

    },
    removeImg(ind) {
      let img = this.imgs[ind];
      if(img.type == 'station') {
        this.stations = [];
      }
      if(img.type == 'reactor') {
        this.reactors = [];
      }
      this.imgs.splice(ind,1);
    },
    mouseOver(block) {
      let templ = this.templates[this.selectedTemplate];
      this.templCover.pos.x = block.pos.x;
      this.templCover.pos.y = block.pos.y;
      this.templCover.size.w = templ.width;
      this.templCover.size.h = templ.height;
      this.templCover.image = templ.image;
    },
    selectBlock(block) {
      // if(this.imgs.length >= this.blockLimit) {
      //   alert('Maximum ' + this.blockLimit + ' blocks on map');
      //   return;
      // }
      this.selectedBlock = block;
      let templ = this.templates[this.selectedTemplate];
      if(templ.type == 'robot_caring' && this.playerCount>0) {
        alert('Maximum 1 robot in map');
        return;
      }
      let pos = {};
      pos["x"] = this.selectedBlock.pos.x;
      pos["y"] = this.selectedBlock.pos.y;
      if(templ.type == 'station') {
        this.stations.push({"type": "station"});
      }
      if(templ.type == 'robot_caring') {
        this.playerCount++;
      }
      if(templ.type == 'exit') {
        this.exitCount++;
      }
      if(templ.type == 'reactor') {
        this.reactors.push({"type": "reactor"});
      }
      this.imgs.push({
        "image": templ.image,
        "pos": pos,
        "type": templ.type,
        "isback": templ.isback
      });
    },
    generateBlocks() {
      for(let i=0;i<this.width;i++) {
        for(let j=0;j<this.height;j++) {
          this.blocks.push({
            "id": i + 'x' + j,
            "pos": {
            "x": (i*32),
            "y": (j*32),
            "i": i+1,
            "j": j+1
            }
          });
        }
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .result-modal-back {
    position: fixed;
    top:0;
    left:0;
    right:0;
    bottom: 0;
    background-color: rgba(0,0,0,0.3);
    z-index: 999;
  }
  .result-modal {
    margin: 50px;
    background-color: aliceblue;
    padding: 25px;
  }
  .map-actions {
    position: absolute;
    top:0;left: 0;right: 0;
    height: 130px;
    background: #2c3e50;
  }
  .map-export-btn {
    background: #42b983;
    color: #2c3e50;
    border-radius: 3px;
    cursor: pointer;
    width: 180px;
    margin: 15px;
    height: 32px;
    line-height: 32px;
  }
  .map-export-info {
    margin-top: 5px;
    color: aliceblue;
  }

.map-builder {
position:absolute;
left: 250px;
top: 0;
}
.templCover {
  position: absolute;
  background: #42b983;
  opacity: 0.3;
  background-size: contain;
  background-position: center;
  background-repeat: no-repeat;
}
.map-list {
  position: fixed;
  top:0;left:0;bottom:0;
  padding-top: 140px;
  width: 230px;
  overflow-y:auto;
  z-index: 99;
  background: aliceblue;
}
.map-list-item {
 position: relative;
 width: 100%;
}
.map-list-item--selected {
background: red;
}

.gameBlock {
position: absolute;
width: 32px;
height: 32px;
background: red;
}
.gridBlock {
position: absolute;
width: 30px;
height: 30px;
border: 1px solid red;
}
.blockImage {
position: absolute;
  cursor:pointer;
}
.gridBlock--selected {
/*background: gray;*/
}
</style>
