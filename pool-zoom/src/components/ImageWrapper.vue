<template>
  <div>
    <div id="canvas" class="background-image-wrapper" @mousemove="onmousemove" @click="onclick($event)">
      <div id="popup-menu" class="menu-box" :class="{'show': showMenuBox}" @click="(e)=>{ e.stopPropagation()}">
        <div class="col-flex-center">
          <span>Select the product SKU:</span>
          <input type="text" class="popup-menu-list-input" @keyup="filterProducts($event)" v-model="inputProductList">
        </div>
        <div class="col-flex-center">
          <div class="popup-menu-list-wrapper" :class="{'show': showProductsList}">
            <div v-for="p in filteredProducts" :key="p.id" class="list-item" @click="addChip(p)">
              {{ p.name }}
            </div>
          </div>
        </div>
        <div class="col-flex-center">
          <span>Type the diagram part number:</span>
          <input type="text" class="popup-menu-list-input" @keyup="keyupDiagramNumber($event)" v-model="inputDiagramNumber">
        </div>
        <div class="wrapper-buttons">
          <button @click="saveHotSpot($event)">Ok</button>
          <button @click="cancelOpenHotSpot($event)">Cancel</button>
        </div>
      </div>
      <div id="popup-delete" class="delete-box" :class="{'show': showDeleteBox}" @click="(e)=>{ e.stopPropagation()}">
        <div>
          Are you sure you want to delete this hotspot ?
        </div>
        <div class="wrapper-buttons">
          <button @click="deleteHotSpot($event)">Yes</button>
          <button @click="cancelDeleteHotSpot($event)">Cancel</button>
        </div>
      </div>
    </div>
    <div v-for="h in listHotSpots" :key="h.id">
        <div v-for="p in h.listProductsHotSpot" :key="p.id">
          Part {{ h.partName }} : {{ JSON.stringify(p) }}
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'ImageWrapper',
  props: {
    msg: String
  },
  data() {
    return {
      filteredProducts: [],
      listHotSpots: [],
      products: [
        {id:1, sku: 'POL-201-6509', name: 'POL-201-6509'},
        {id:2, sku: 'POL-201-6510', name: 'POL-201-6510'},
        {id:3, sku: 'POL-201-6511', name: 'POL-201-6511'},
        {id:4, sku: 'POL-201-6512', name: 'POL-201-6512'},
        {id:5, sku: 'POL-201-6513', name: 'POL-201-6513'},
        {id:6, sku: 'POL-201-6514', name: 'POL-201-6514'},
        {id:7, sku: 'POL-201-6525', name: 'POL-201-6525'},
        {id:8, sku: 'POL-201-6526', name: 'POL-201-6526'},
        {id:9, sku: 'POL-201-6537', name: 'POL-201-6537'},
        {id:10, sku: 'POL-201-6538', name: 'POL-201-6538'},
      ],
      inputProductList: '',
      inputDiagramNumber: '',
      showProductsList: false,
      count: 0,
      mouse: null,
      element: null,
      showMenuBox: false,
      showDeleteBox: false,
      selectedId: null,
      timeout: null,
    }
  },
  methods:{
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
      this.showDeleteBox = false;
      this.showMenuBox = false;
      this.showProductsList = false;
        if (this.element !== null) {
            this.element.id = 'box-'+this.count++;
            let id = this.element.id;
            this.listHotSpots.push({id: id, partName: '', listProductsHotSpot:[]});
            this.element.addEventListener("click", e => this.clickHotSpot(e, id));
            this.element.addEventListener("dblclick", e => this.dblClickHotSpot(e, id));
            this.dragElement(document.getElementById(id));
            this.element = null;
            canvas.style.cursor = "default";
        } else {
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
    clickHotSpot(e, idHotSpot){
      this.selectedId = idHotSpot;
      e.stopPropagation();
      clearTimeout(this.timeout);
      this.timeout = setTimeout(() => {
        this.openHotSpotMenu(idHotSpot);
      }, 500);
    },
    openHotSpotMenu(idHotSpot){
      this.inputProductList = '';
      let index = this.findIndexArrById(this.listHotSpots, this.selectedId);
      if(index != -1){
        this.inputDiagramNumber = this.listHotSpots[index].partName;
      }else{
        this.inputDiagramNumber = '';
      }
      this.showMenuBox = true;
      this.showDeleteBox = false;
    },
    dblClickHotSpot(e, idHotSpot){
      clearTimeout(this.timeout);
      this.showDeleteBox = true;
      this.showMenuBox = false;
      this.selectedId = idHotSpot;
    },
    saveHotSpot(e){
      e.stopPropagation();
      this.showMenuBox = false;
      this.inputProductList = '';
      let index = this.findIndexArrById(this.listHotSpots, this.selectedId);
      if(index != -1){
        this.inputDiagramNumber = this.listHotSpots[index].partName;
      }else{
        this.inputDiagramNumber = '';
      }
    },
    cancelOpenHotSpot(e){
      e.stopPropagation();
      this.showMenuBox = false;
      this.inputProductList = '';
      this.inputDiagramNumber = '';
    },
    deleteHotSpot(e){
      e.stopPropagation();
      this.showDeleteBox = false;
      let selectedHotSpot = document.getElementById(this.selectedId);
      this.removeHotSpotById(this.listHotSpots, this.selectedId);
      selectedHotSpot.remove();
    },
    cancelDeleteHotSpot(e){
      e.stopPropagation();
      this.showDeleteBox = false;
    },
    dragElement(elmnt) {
        var pos1 = 0, pos2 = 0, pos3 = 0, pos4 = 0;
        elmnt.onmousedown = dragMouseDown;

        function dragMouseDown(e) {
          e = e || window.event;
          e.preventDefault();
          
          // get the mouse cursor position at startup:
          pos3 = e.clientX;
          pos4 = e.clientY;
          document.onmouseup = closeDragElement;
          // call a function whenever the cursor moves:
          document.onmousemove = elementDrag;
          e.stopPropagation();
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
          e.stopPropagation();
        }

        function closeDragElement(e) {
          /* stop moving when mouse button is released:*/
          document.onmouseup = null;
          document.onmousemove = null;
          
        }
    },
    filterProducts(event){
      // TO DO : ajax call
      this.showProductsList = false;
      if(event.target.value && event.target.value != '' && event.target.value.length > 1) {
          this.showProductsList = true;
          this.filteredProducts = this.products.filter( (item) => {
            return item.name.toLowerCase().includes(event.target.value.toLowerCase());
          });
      }else{
          this.showProductsList = false;
          this.filteredProducts = [];
          this.filteredProducts.push(...this.products);
      }
    },
    keyupDiagramNumber(event){
      let index = this.findIndexArrById(this.listHotSpots, this.selectedId);
      if(index != -1){
        this.listHotSpots[index].partName = this.inputDiagramNumber;
      }
    },
    addChip(p){
      let index = this.findIndexArrById(this.listHotSpots, this.selectedId);
      if(index != -1){
        this.listHotSpots[index].listProductsHotSpot.push(p);
      }
    },
    findIndexArrById(arr, id){
      let found = -1;
      arr.forEach((element, i) => {
        if(element.id  == id){
          found = i;
        }
      });
      return found;
    },
    removeHotSpotById(arr, id){
      let index = -1;
      arr.forEach((element, i) => {
        if(element.id  == id){
          index = i;
        }
      });
      if(index != -1){
        arr.splice(index, 1);
      }
    }
  },
  mounted: function(){
    let canvas = document.getElementById('canvas');
    this.initDraw(canvas);
    this.filteredProducts = this.products;
  },
}
</script>
