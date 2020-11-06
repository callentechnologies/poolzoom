<template>
  <div id="canvas" class="background-image-wrapper" @mousemove="onmousemove" @click="onclick($event)">
    <!-- <div v-for="field in fields" :key="field.id" class="hot-spot-wrapper" v-bind="validationRules(field)"  v-bind:style="styleRules(field)">
    </div> -->
  </div>
</template>

<script>

// Vue.component('CompnentTest', {
//   data() {
//     return {
//       text: 'some text inside the header'
//     }
//   },
//   render(createElement) {
//     return createElement('h1', this.text)
//   }
// });

export default {
  name: 'ImageWrapper',
  props: {
    msg: String
  },
  data() {
    return {
      fields: [],
      count: 0,
      mouse: null,
      element: null,
      createBoxState: null,
    }
  },

  methods:{
    addFormElement: function(type, coordX, coordY) {
      console.log('addFormElement');
      this.fields.push({
        'id': type+this.count,
        'type': type,
        'x': coordX,
        'y': coordY,
      });
    },
    styleRules(field){
      return {
        left: field.x+'px',
        top: field.y+'px',
      }
    },
    validationRules(field) {
      return {
        id: field.id,
        type: field.type
      }
    },

    initDraw(canvas) {
        this.mouse = {
            x: 0,
            y: 0,
            startX: 0,
            startY: 0
        };
        this.element = null;
    },
    setMousePosition: function(e) {
      var ev = e || window.event;
      if (ev.pageX) { 
          this.mouse.x = ev.pageX + window.pageXOffset;
          this.mouse.y = ev.pageY + window.pageYOffset;
      }
    },
    onmousemove: function (e) {
        this.setMousePosition(e);
        if (this.element !== null) {
            this.element.style.width = Math.abs(this.mouse.x - this.mouse.startX) + 'px';
            this.element.style.height = Math.abs(this.mouse.y - this.mouse.startY) + 'px';
            this.element.style.left = (this.mouse.x - this.mouse.startX < 0) ? this.mouse.x + 'px' : this.mouse.startX + 'px';
            this.element.style.top = (this.mouse.y - this.mouse.startY < 0) ? this.mouse.y + 'px' : this.mouse.startY + 'px';
        }
    },
    onclick: function (e) {
        if (this.element !== null) {
            this.element.id = 'box-'+this.count;
            this.element.onclick = this.clickBox(e);
            console.log("B finsihed.", this.element);
            this.element = null;
            canvas.style.cursor = "default";
            this.createBoxState = 'finsihed';
            console.log("finsihed.");
        } else {
            console.log("begun.");
            this.createBoxState = 'begun';
            this.mouse.startX = this.mouse.x;
            this.mouse.startY = this.mouse.y;
            this.element = document.createElement('div');
            
            this.element.className = 'rectangle'
            this.element.style.left = this.mouse.x + 'px';
            this.element.style.top = this.mouse.y + 'px';
            canvas.appendChild(this.element);
            // canvas.style.cursor = "crosshair";
        }
    },
    clickBox: function(e){
      console.log('clickBox', e);
      if(this.createBoxState == 'begun'){
      }else{
        console.log('stopImmediatePropagation');
        e.stopImmediatePropagation()();
      }
    },


    dragElement(elmnt) {
      console.log('elmnt', elmnt);
        var pos1 = 0, pos2 = 0, pos3 = 0, pos4 = 0;
        if (document.getElementById(elmnt.id + "header")) {
          /* if present, the header is where you move the DIV from:*/
          document.getElementById(elmnt.id + "header").onmousedown = dragMouseDown;
        } else {
          /* otherwise, move the DIV from anywhere inside the DIV:*/
          elmnt.onmousedown = dragMouseDown;
        }

        function dragMouseDown(e) {
          e = e || window.event;
          e.preventDefault();
          // get the mouse cursor position at startup:
          pos3 = e.clientX;
          pos4 = e.clientY;
          document.onmouseup = closeDragElement;
          // call a function whenever the cursor moves:
          document.onmousemove = elementDrag;
        }

        function elementDrag(e) {
          e = e || window.event;
          e.preventDefault();
          // calculate the new cursor position:
          pos1 = pos3 - e.clientX;
          pos2 = pos4 - e.clientY;
          pos3 = e.clientX;
          pos4 = e.clientY;
          // set the element's new position:
          elmnt.style.top = (elmnt.offsetTop - pos2) + "px";
          elmnt.style.left = (elmnt.offsetLeft - pos1) + "px";
        }

        function closeDragElement() {
          /* stop moving when mouse button is released:*/
          document.onmouseup = null;
          document.onmousemove = null;
        }
      }
  },
  mounted: function(){
    let canvas = document.getElementById('canvas');
    this.initDraw(canvas);

    console.log(Vue);
  },
}
</script>
