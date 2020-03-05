<!DOCTYPE html>

<template>
  <html>

  <head>
    <title>Bootstrap Example</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.0/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.8.0/css/all.css"
          integrity="sha384-Mmxa0mLqhmOeaE8vgOSbKacftZcsNYDjQzuCOm6D02luYSzBG8vpaOykv9lFQ51Y" crossorigin="anonymous">
    <!--    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>-->
    <!--    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>-->

    <!--    <script src="vue.js"></script>-->
  </head>

  <body>

  <!-- Main container-->
  <div class="container-fluid"
       style="padding-right:0; padding-left:0; height: 100%; display: flex; flex-direction: column;">
    <div class="sticky-top">
      <nav class="navbar sticky-top navbar-expand-sm navbar-dark bg-dark">
        <!--        <a class="navbar-brand" href="#"></a>-->
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent"
                aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <ul class="navbar-nav mr-auto">

            <li class="nav-item active">
              <button class="btn btn-outline-success navbar" type="button" @click="solve()">Start</button>
            </li>

            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown"
                 aria-haspopup="true" aria-expanded="false">
                Size
              </a>
              <div class="dropdown-menu" aria-labelledby="navbarDropdown" id="sizeOptions">
                <b-dropdown-item class="nav-link active" @click="setSize(1)">30x50</b-dropdown-item>
                <b-dropdown-item class="nav-link" @click="setSize(2)">40x66</b-dropdown-item>
                <b-dropdown-item class="nav-link" @click="setSize(3)">50x83</b-dropdown-item>
                <b-dropdown-item class="nav-link" @click="setSize(4)">60x100</b-dropdown-item>
              </div>
            </li>

            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown"
                 aria-haspopup="true" aria-expanded="false">
                Alogorithm
              </a>
              <div class="dropdown-menu" aria-labelledby="navbarDropdown">
                <b-dropdown-item class="nav-link active">Breadthfirst</b-dropdown-item>
                <b-dropdown-item class="nav-link disabled">Depthfirst</b-dropdown-item>
              </div>
            </li>

            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown"
                 aria-haspopup="true" aria-expanded="false">
                Reset Grid
              </a>
              <div class="dropdown-menu" aria-labelledby="navbarDropdown">
                <b-dropdown-item @click="clearGrid(false)">Reset (keep walls)</b-dropdown-item>
                <b-dropdown-item @click="clearGrid(true)">Reset (clear walls)</b-dropdown-item>
              </div>
            </li>

          </ul>
        </div>
      </nav>
    </div>
    <!-- Modal -->
    <transition name="modal" mode="out-in">
      <div v-if="showModalInstructions" key="warning">
        <div class="modal-mask">
          <div class="modal-wrapper" @click="showModalInstructions=false">
            <div class="modal-container" @click.stop>
              <div class="modal-header">
                <slot name="header">
                  Instructions
                </slot>
              </div>
              <div class="modal-body">
                <ol class="list-group list-group-flush">
                  <li class="list-group-item">Select a start and end point by right clicking a tile.</li>
                  <li class="list-group-item">Draw walls by clicking and dragging over tiles.</li>
                  <li class="list-group-item">Select an algorithm.</li>
                  <li class="list-group-item">Hit start!</li>
                </ol>
              </div>
              <div style="text-align:center">
                <slot>
                  <button class="btn btn-outline-dark btn-lg btn-block" @click="showModalInstructions=false">
                    Got it!
                  </button>
                </slot>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- Modal -->
    <transition name="modal" mode="out-in">
      <div v-if="showModalNoPath" key="warning">
        <div class="modal-mask">
          <div class="modal-wrapper" @click="showModalNoPath=false">
            <div class="modal-container" @click.stop>
              <div class="modal-header">
                <slot name="header">
                  No path was found!
                </slot>
              </div>
              <div class="modal-body">
              </div>
              <div style="text-align:center">
                <slot>
                  <button class="btn btn-outline-dark btn-lg btn-block" @click="clearGrid(true)">
                    Clear Grid
                  </button>
                </slot>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <ul class='custom-menu'>
      <li @click="setStart()" data-action="first">Set Start Tile</li>
      <li @click="setEnd()" data-action="second">Set End Tile</li>
    </ul>
    <!--    Grid-->
    <div id="grid-container"></div>
    <div id="inner_remaining"></div>
  </div>

  <!--  <div class="inner_remaining" style="flex: 1 1 auto; background-color: black"></div>-->
  </body>
  <footer>
  </footer>
  </html>
</template>

<script>
  import $ from 'jquery'

  let isDrawing = false;
  let rows = 30;
  let cols = 50;
  let startPos = [15, 40];
  let targetPos = [20, 20];
  let rightClickID = "";
  let isRunning = false;

  //Prepare Components
  function createGrid(rows, cols) {
    const container = document.getElementById("grid-container");
    container.style.setProperty('--grid-rows', rows);
    container.style.setProperty('--grid-cols', cols);
    for (let i = 0; i < (rows * cols); i++) {
      let cell = document.createElement("div");
      container.appendChild(cell).className = "grid-item";
      cell.id = "cell-" + i;
      cell.ondragstart = "return false;";
      cell.ondrop = "return false;";
    }
    startPos = [15, 40];
    targetPos = [20, 20];
    let cell = document.getElementById("cell-" + getNodeIndex(startPos[0], startPos[1]));
    cell.style.backgroundColor = "red";
    cell = document.getElementById("cell-" + getNodeIndex(targetPos[0], targetPos[1]));
    cell.style.backgroundColor = "green";
  }

  function clearGrid(eraseWalls) {
    for (let i = 0; i < (rows * cols); i++) {
      let cell = document.getElementById("cell-" + i);
      if (cell.style.animation || cell.style.animation.toString().length !== 0) {
        cell.style.animation = 'none';
        cell.offsetHeight; /* trigger reflow */
        cell.style.animation = null;
        cell.style.backgroundColor = "white";
      }
      if (eraseWalls && cell.style.backgroundColor !== "green" && cell.style.backgroundColor !== "red") {
        cell.style.backgroundColor = "white";
      }
    }
  }

  function drawWall(id) {
    let cell = document.getElementById(id);
    if (cell.style.backgroundColor !== "green" && cell.style.backgroundColor !== "red") {
      if (cell.style.animation || cell.style.animation.toString().length !== 0) {
        cell.style.animation = 'none';
        cell.offsetHeight; /* trigger reflow */
        cell.style.animation = null;
      }
      cell.style.backgroundColor = "black";
    }
  }

  function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  function getNodeIndex(row, col) {
    if (row >= 0 && col >= 0 && row < rows && col < cols) {
      return (row * cols) + col;
    }
    return null;
  }

  function getAdjNode(position) {
    let index = getNodeIndex(position[0], position[1]);
    if (position[0] < 0 || position[0] >= rows || position[1] < 0 || position[1] >= cols) {
      return null;
    } else if (document.getElementById("cell-" + index).style.backgroundColor === "black") {
      return null;
    } else {
      return position;
    }
  }

  function createGridGraph() {
    let gridGraph = [];
    for (let i = 0; i < rows; i++) {
      for (let j = 0; j < cols; j++) {
        let right = getAdjNode([i, j + 1]);
        let left = getAdjNode([i, j - 1]);
        let up = getAdjNode([i - 1, j]);
        let down = getAdjNode([i + 1, j]);
        let node = {value: [i, j], left: left, right: right, up: up, down: down};
        gridGraph.push(node);
      }
    }
    return gridGraph;
  }

  function bfs() {
    let visited = [];
    let gridGraph = createGridGraph();
    let queue = [];
    let visitOrder = [];
    let path = [];
    let targetIndex = getNodeIndex(targetPos[0], targetPos[1]);
    let startIndex = getNodeIndex(startPos[0], startPos[1]);
    let node = gridGraph[startIndex];
    let targetNode = gridGraph[targetIndex];
    visited [startIndex] = true;
    path.push(node);
    queue.push(path);

    while (queue.length !== 0) {
      path = queue.shift();
      node = path[path.length - 1];
      visitOrder.push(node);

      if (node === targetNode) {
        return [path, visitOrder];
      }

      if (node.left && !visited[getNodeIndex(node.left[0], node.left[1])]) {
        let leftIndex = getNodeIndex(node.left[0], node.left[1]);
        let pathToNextNode = path.slice();
        pathToNextNode.push(gridGraph[leftIndex]);
        queue.push(pathToNextNode);
        visited[leftIndex] = true;
      }
      if (node.right && !visited[getNodeIndex(node.right[0], node.right[1])]) {
        let rightIndex = getNodeIndex(node.right[0], node.right[1]);
        let pathToNextNode = path.slice();
        pathToNextNode.push(gridGraph[rightIndex]);
        queue.push(pathToNextNode);
        visited[rightIndex] = true;
      }
      if (node.up && !visited[getNodeIndex(node.up[0], node.up[1])]) {
        let upIndex = getNodeIndex(node.up[0], node.up[1]);
        let pathToNextNode = path.slice();
        pathToNextNode.push(gridGraph[upIndex]);
        queue.push(pathToNextNode);
        visited[upIndex] = true;
      }
      if (node.down && !visited[getNodeIndex(node.down[0], node.down[1])]) {
        let downIndex = getNodeIndex(node.down[0], node.down[1]);
        let pathToNextNode = path.slice();
        pathToNextNode.push(gridGraph[downIndex]);
        queue.push(pathToNextNode);
        visited[downIndex] = true;
      }
    }
    return null;
  }

  //Wait fo document to create initial grid
  $(document).ready(function () {
    createGrid(rows, cols);
  });

  // Add mouse listeners to draw
  $(document).on('mousedown mousemove mouseup dragover dragleave', '.grid-item', function (e) {
    if (e.which === 1 && !isRunning) {
      if (e.type === "mousedown" || e.type === "dragover") {
        isDrawing = true;
        drawWall(e.target.id);
      } else if (e.type === "mousemove" && isDrawing) {
        drawWall(e.target.id);
      } else if (e.type === "mouseup" || e.type === "dragleave") {
        isDrawing = false;
      }
    }
  });

  // Trigger action when the contexmenu is about to be shown
  $(document).bind("contextmenu", function (event) {
    // Avoid the real one
    event.preventDefault();
    if (!isRunning && event.target.id.toString().includes("cell-")) {
      rightClickID = event.target.id;
      // Show contextmenu
      $(".custom-menu").finish().toggle(100).
        // In the right position (the mouse)
        css({
          top: event.pageY + "px",
          left: event.pageX + "px"
        });
    }
  });

  // If the document is clicked somewhere
  $(document).bind("mousedown", function (e) {

    // If the clicked element is not the menu
    if (!$(e.target).parents(".custom-menu").length > 0) {

      // Hide it
      $(".custom-menu").hide(100);
    }
  });
  export default {
    name: 'home',
    data() {
      return {
        msg: 'Example',
        showModalInstructions: true,
        showModalNoPath: false
      }
    },
    created() {


    },
    mounted() {
      let jqueryScript = document.createElement('script')
      jqueryScript.setAttribute('src', 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js')
      document.head.appendChild(jqueryScript)

      let bootStrapScript = document.createElement('script')
      bootStrapScript.setAttribute('src', 'https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js')
      document.head.appendChild(bootStrapScript)


    },

    methods: {
      solve: async function () {
        let pathAndVisitOrder = bfs();
        if (!pathAndVisitOrder) {
          this.showModalNoPath = true;
        } else if (!isRunning) {
          clearGrid(false);
          isRunning = true;
          let visitedOrder = pathAndVisitOrder[1];
          let path = pathAndVisitOrder[0];
          for (let i = 0; i < visitedOrder.length; i++) {
            let node = visitedOrder[i];
            let nodeIndex = getNodeIndex(node.value[0], node.value[1]);
            let cell = document.getElementById("cell-" + nodeIndex);
            if (cell.style.backgroundColor !== "green" && cell.style.backgroundColor !== "red") {
              await sleep(5);
              cell.style.animation = "mymove 5s";
              cell.style.animationFillMode = "both";
            }
          }
          for (let i = 0; i < path.length; i++) {
            let node = path[i];
            let nodeIndex = getNodeIndex(node.value[0], node.value[1]);
            let cell = document.getElementById("cell-" + nodeIndex);
            if (cell.style.backgroundColor !== "green" && cell.style.backgroundColor !== "red") {
              await sleep(5);
              cell.style.animation = "mymove-1 4s";
              cell.style.animationFillMode = "both";
            }
          }
          isRunning = false;
        }
      },
      clearGrid: function (eraseWalls) {
        if (isRunning) {
          return;
        }
        clearGrid(eraseWalls);
        this.showModalNoPath = false;
      },
      setStart: function () {
        $(".custom-menu").hide(100);
        let cell = document.getElementById("cell-" + getNodeIndex(startPos[0], startPos[1]));
        cell.style.backgroundColor = "white";
        cell = document.getElementById(rightClickID);
        cell.style.backgroundColor = "red";
        let id = rightClickID.substring(5);
        let row = Math.floor(id / cols);
        let col = id % cols;
        startPos = [row, col];
      },
      setEnd: function () {
        $(".custom-menu").hide(100);
        let cell = document.getElementById("cell-" + getNodeIndex(targetPos[0], targetPos[1]));
        cell.style.backgroundColor = "white";
        cell = document.getElementById(rightClickID);
        cell.style.backgroundColor = "green";
        let id = rightClickID.substring(5);
        let row = Math.floor(id / cols);
        let col = id % cols;
        targetPos = [row, col];
      },
      setSize: function (size) {
        if (isRunning) {
          return;
        }

        // Get the container element
        var sizeContainer = document.getElementById("sizeOptions");
        // Get all buttons with class="btn" inside the container
        var sizeOptions = sizeContainer.getElementsByClassName("nav-link");
        var selected = sizeOptions[size - 1];
        // Loop through the buttons and add the active class to the current/clicked button
        for (var i = 0; i < sizeOptions.length; i++) {
          var current = sizeOptions[i];
          current.className = current.className.replace(" active", "");

        }
        selected.className += " active";
        const container = document.getElementById("grid-container");
        container.innerHTML = '';
        switch (size) {
          case 1:
            rows = 30;
            cols = 50;
            break;
          case 2:
            rows = 40;
            cols = 66;
            break;
          case 3:
            rows = 50;
            cols = 83;
            break;
          case 4:
            rows = 60;
            cols = 100;
            break;
          default:
            break;
        }
        createGrid(rows, cols);

      }
    }
  }

</script>

<style>

  h1, h2 {
    font-weight: normal;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }

  :root {
    --grid-cols: 1;
    --grid-rows: 1;
  }

  .btn-outline-dark {
    color: gray;
  }

  #grid-container {
    display: grid;
    grid-gap: 0cm;
    border-left: 1px solid black;
    border-top: 1px solid black;
    grid-template-rows: repeat(var(--grid-rows), 1fr);
    grid-template-columns: repeat(var(--grid-cols), 1fr);
    margin-bottom: 0px;
    margin-right: 0px;
    margin-left: 0px;
    alignment: center;
  }

  #ranks li {
    display: inline-block;
  }

  .grid-item {
    width: 100%;
    border-right: 1px solid black;
    border-bottom: 1px solid black;
    text-align: center;
    background-color: white;
  }


  .grid-item:after {
    content: "";
    display: block;
    padding-bottom: 100%;
  }

  #img-container:hover img {
    opacity: 0.8;
  }

  .btn-outline-dark:hover {
    color: white !important;
    cursor: pointer;
  }

  .navbar {
    margin-bottom: 0 !important;
  }


  @keyframes mymove {
    from {
      background-color: orange;
    }
    to {
      background-color: blue;
    }
  }

  @keyframes mymove-1 {
    from {
      background-color: orange;
    }
    to {
      background-color: yellow;
    }
  }

  footer {
    background-color: black;
    height: 100%;
  }

  /*transition, modal, modal-mask, modal wrapper, modal container, (header, bodt, footer)*/
  .modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, .5);
    display: table;
    transition: opacity .3s ease;
    overflow-y: scroll;
    vertical-align: top;
  }

  .modal-wrapper {
    position: relative;
    display: table-cell;
    margin-top: 1.75cm;
    margin-bottom: 1.75cm;
    vertical-align: top;
  }

  .modal-container {
    margin: auto;
    margin-top: 1.75cm;
    margin-bottom: 1.75cm;
    width: 400px;
    /*margin: 0px auto;*/
    padding: 20px 30px;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, .33);
    transition: all .3s ease;
    font-family: Helvetica, Arial, sans-serif;
    display: flex;
    flex-direction: column;
  }

  .modal-header {
    margin-top: 0;
    font-size: 24px;
  }

  .modal-body {
    overflow-y: auto;
    max-height: calc(100vh - 200px);
    flex: 1 1 auto;
    margin: 0;
  }


  .modal-default-button {
    float: right;
  }

  .modal-enter {
    opacity: 0;
  }

  .modal-leave-active {
    opacity: 0;
    transition: all .3s ease;
  }

  /* The whole thing */
  .custom-menu {
    display: none;
    z-index: 1000;
    position: absolute;
    overflow: hidden;
    border: 1px solid #CCC;
    white-space: nowrap;
    font-family: sans-serif;
    background: #FFF;
    color: #333;
    border-radius: 5px;
    padding: 0;
  }

  /* Each of the items in the list */
  .custom-menu li {
    padding: 8px 12px;
    cursor: pointer;
    list-style-type: none;
    transition: all .3s ease;
    user-select: none;
    display: block;
  }

  html, body, #full {
    position: absolute;
    top: 0px; /* Header Height */
    bottom: 0px; /* Footer Height */
    width: 100%;
  }

  .custom-menu li:hover {
    background-color: #DEF;
  }

  #inner_remaining {
    height: 100%;
    flex: 1;
    display: flex;
    flex-direction: column;
    background-color: #343a40 !important;
  }

  .modal-enter .modal-container,
  .modal-leave-active .modal-container,
  .modal-leave-to .modal-container {
    /*-webkit-transform: scale(1.1);*/
    transform: translateY(100px);
  }
</style>
