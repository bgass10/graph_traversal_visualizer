<script>
  import P5 from "p5-svelte";
  const clickedRun = () => {
    for (let i = 0; i < nodes.length; i++) {
      nodes[i].visited = false;
    }
    if (algo == "BFS") {
      bfs();
    } else if (algo == "DFS") {
      dfs();
    }
  };
  const bfs = () => {
    res = [];
    var beginPoint = startNode;
    startNode = -1;
    var q = [];
    var visited = new Set();
    q.push(beginPoint);
    visited.add(beginPoint);
    while (q.length > 0) {
      var vertex = q.shift();
      res.push(vertex);
      var adjacent = graph[vertex];
      for (var v of adjacent) {
        if (!visited.has(v)) {
          visited.add(v);
          q.push(v);
        }
      }
    }
    runAlgo = true;
    console.log(res);
  };
  const dfs = () => {
    res = [];
    var beginPoint = startNode;
    startNode = -1;
    var visited = new Set();
    dfs_helper(beginPoint, visited);
    runAlgo = true;
    console.log(res);
  };
  const dfs_helper = (vertex, visited) => {
    visited.add(vertex);
    res.push(vertex);
    for (var neighbor of graph[vertex]) {
      if (!visited.has(neighbor)) {
        dfs_helper(neighbor, visited);
      }
    }
  };

  const insideNode = (x, y) => {
    for (var node of nodes) {
      if (
        x > node.x - d / 2 &&
        x < node.x + d / 2 &&
        y > node.y - d / 2 &&
        y < node.y + d / 2
      ) {
        return node;
      }
    }
    return -1;
  };
  const placeEdge = (p5, node) => {
    p5.stroke("#22D1EE");
    p5.strokeWeight(3);
    p5.line(node.x, node.y, p5.mouseX, p5.mouseY);
  };
  const placeNode = (p5) => {
    p5.noStroke();
    p5.fill("#3D5AF1");
    p5.ellipse(p5.mouseX, p5.mouseY, d);
    p5.fill("white");
    p5.text(nodes.length.toString(), p5.mouseX, p5.mouseY);
  };
  const clickedAddNode = () => {
    if (!placing) {
      placing = true;
      addNodeText = "x";
    } else {
      placing = false;
      addNodeText = "+";
    }
  };
  const clickedAddEdge = () => {
    if (!placingEdge) {
      placingEdge = true;
      addEdgeText = "x";
    } else {
      placingEdge = false;
      addEdgeText = "+";
    }
  };
  const animateAlgo = (p5) => {
    timer--;
    if (timer <= 0 && currNodeIdx < res.length - 1) {
      currNodeIdx++;
      timer = 120;
    } else if (timer <= 0) {
      runAlgo = false;
      timer = 120;
      currNodeIdx = 0;
    }
    if (runAlgo) {
      p5.fill("#ff8a27");
      p5.ellipse(nodes[res[currNodeIdx]].x, nodes[res[currNodeIdx]].y, d);
      p5.fill("white");
      p5.text(
        nodes[res[currNodeIdx]].id,
        nodes[res[currNodeIdx]].x,
        nodes[res[currNodeIdx]].y
      );
      nodes[res[currNodeIdx]].visited = true;
    }
  };
  var graph = {};
  var d = 60;
  var nodes = [];
  var edges = [];
  var placing = false;
  var placingEdge = false;
  var startNode = -1;
  var algo;
  var addNodeText = "+";
  var addEdgeText = "+";
  var haveFirstAnchor = false;
  var clickedNode = -1;
  var clickedNode2 = -1;
  var currentNode = -1;
  var runAlgo = false;
  var res = [];
  var timer = 120;
  var currNodeIdx = 0;
  var allNodes = [];

  const sketch = (p5) => {
    p5.setup = () => {
      var canvas = p5.createCanvas(p5.windowWidth, p5.windowHeight - 80);
      p5.textAlign(p5.CENTER, p5.CENTER);
      p5.textSize(24);
    };

    p5.draw = () => {
      p5.background("#E2F3F5");
      if (startNode != -1) {
        p5.noFill();
        p5.strokeWeight(4);
        p5.stroke("#3D5AF1");
        p5.ellipse(nodes[startNode].x, nodes[startNode].y, d);
        p5.noStroke();
      }
      for (let i = 0; i < edges.length; i++) {
        p5.stroke("#22D1EE");
        p5.strokeWeight(3);
        p5.line(
          edges[i].start[0],
          edges[i].start[1],
          edges[i].end[0],
          edges[i].end[1]
        );
      }
      for (let i = 0; i < nodes.length; i++) {
        p5.fill("#22D1EE");
        p5.noStroke();
        p5.ellipse(nodes[i].x, nodes[i].y, d);
      }
      if (placing) {
        placeNode(p5);
      }
      if (haveFirstAnchor) {
        placeEdge(p5, clickedNode);
      }
      for (let i = 0; i < nodes.length; i++) {
        p5.fill("white");
        p5.text(nodes[i].id, nodes[i].x, nodes[i].y);
      }
      for (let i = 0; i < nodes.length; i++) {
        p5.strokeWeight(3);
        p5.stroke("#ff8a27");
        p5.fill("white");
        if (nodes[i].visited) {
          p5.ellipse(nodes[i].x, nodes[i].y, d);
          p5.fill("#ff8a27");
          p5.text(nodes[i].id, nodes[i].x, nodes[i].y);
        }
      }
      if (runAlgo) {
        animateAlgo(p5);
      }
    };
    p5.mouseClicked = () => {
      if (haveFirstAnchor && (p5.mouseY > 130 || p5.mouseX > 180)) {
        var clickedX = p5.mouseX;
        var clickedY = p5.mouseY;
        clickedNode2 = insideNode(clickedX, clickedY);
        if (clickedNode2 != -1) {
          haveFirstAnchor = false;
          edges = [
            ...edges,
            {
              start: [clickedNode.x, clickedNode.y],
              end: [clickedNode2.x, clickedNode2.y],
            },
          ];
          graph[clickedNode.id].push(clickedNode2.id);
          graph[clickedNode2.id].push(clickedNode.id);
          clickedNode = -1;
          clickedNode2 = -1;
        } else {
          haveFirstAnchor = false;
          clickedNode = -1;
          clickedNode2 = -1;
        }
      } else if (placing && (p5.mouseY > 130 || p5.mouseX > 180)) {
        graph[nodes.length] = [];
        nodes = [
          ...nodes,
          {
            x: p5.mouseX,
            y: p5.mouseY,
            id: nodes.length.toString(),
            visited: false,
          },
        ];
        placing = false;
        addNodeText = "+";
      } else if (placingEdge && (p5.mouseY > 130 || p5.mouseX > 180)) {
        var clickedX = p5.mouseX;
        var clickedY = p5.mouseY;
        clickedNode = insideNode(clickedX, clickedY);
        if (clickedNode != -1) {
          haveFirstAnchor = true;
        }
      }
    };
  };
</script>

<svelte:head>
  <link
    href="https://fonts.googleapis.com/css2?family=Montserrat:wght@700&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<div>
  <div class="add-node">
    <h2>Add Node</h2>
    <button
      on:click={() => {
        clickedAddNode();
      }}
      class="add-button">{addNodeText}</button
    >
  </div>
  <div class="add-edge">
    <h2>Add Edge</h2>
    <button
      on:click={() => {
        clickedAddEdge();
      }}
      class="add-button">{addEdgeText}</button
    >
  </div>
  <h1 class="anchor">Graph Traversal Visualizer</h1>
  <div class="row">
    <label for="starting node">Starting Node</label>
    <select bind:value={startNode} class="start-select" name="starting node">
      {#each nodes as node}
        <option value={node.id}>{node.id}</option>
      {/each}
    </select>
    <label for="algorithm">Algorithm</label>
    <select bind:value={algo} class="algo-select" name="algorithm">
      <option value="DFS">DFS</option>
      <option value="BFS">BFS</option>
    </select>
    <button on:click={() => clickedRun()} class="run-button">Run</button>
  </div>
</div>

<P5 {sketch} />

<style>
  .anchor {
    position: fixed;
    left: -12px;
    top: -10px;
  }
  .row {
    display: flex;
    flex-direction: row;
    background-color: #0e153a;
    height: 80px;
    align-items: center;
    justify-content: end;
  }
  h1 {
    color: #fff;
    font-size: 36px;
    padding-left: 50px;
    font-family: Montserrat;
  }
  label {
    color: #fff;
    font-size: 24px;
    font-family: Montserrat;
    margin-left: 40px;
  }
  select {
    border-radius: 15px;
    color: #0e153a;
    font-family: Montserrat;
    font-size: 16px;
    margin-top: 10px;
    margin-left: 10px;
  }
  .start-select {
    width: 60px;
    height: 40px;
  }
  .algo-select {
    width: 80px;
    height: 40px;
    border-radius: 15px;
  }
  .run-button {
    width: 100px;
    height: 50px;
    color: white;
    background-color: #3d5af1;
    border-radius: 15px;
    font-family: Montserrat;
    font-size: 32px;
    border-width: 0;
    margin-top: 15px;
    padding: 0;
    margin-right: 30px;
    margin-left: 50px;
  }
  .add-node {
    display: flex;
    flex-direction: row;
    position: fixed;
    width: 123px;
    height: 40px;
    background-color: rgb(207, 207, 207);
    left: 20px;
    top: 100px;
    border-radius: 40px;
    align-items: center;
    justify-content: end;
  }
  .add-edge {
    display: flex;
    flex-direction: row;
    position: fixed;
    width: 123px;
    height: 40px;
    background-color: rgb(207, 207, 207);
    left: 20px;
    top: 160px;
    border-radius: 40px;
    align-items: center;
    justify-content: end;
  }
  h2 {
    color: #0e153a;
    font-family: Montserrat;
    font-size: 12px;
  }
  .add-button {
    width: 30px;
    height: 30px;
    background-color: #3d5af1;
    color: #fff;
    border-radius: 100%;
    margin-right: 8px;
    margin-left: 8px;
    text-align: center;
    margin-top: 8px;
    padding-top: 0;
    padding-bottom: 12px;
  }
</style>
