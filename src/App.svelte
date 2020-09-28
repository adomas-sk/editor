<script lang="ts">
  import { onMount } from 'svelte';
  import { update } from 'ramda';
  
  import { elements } from './store';
  
  onMount(() => elements.update(els => [
    ...els,
    { x: 10, y: 10, type: 'sidebar', id: Math.random() },
    { x: 500, y: 500, type: 'details', id: Math.random(), visible: false },
  ]));
  
  let elementsToRender = [];
  elements.subscribe(els => {
    elementsToRender = els;
  });
  
  let activeContainer = '';
  
  let isDraging = false;
  
  let prevMouseX = 0;
  let prevMouseY = 0;
  let indexInStore = 0;
  
  let updatePosition: any = () => {};
  
  const mousedownHandler = (currentMouseX, currentMouseY, updatePositionHandler, index, elementId?) => {
    if (elementId) {
      activeContainer = elementId;
      
    }
    isDraging = true;
    prevMouseX = currentMouseX;
    prevMouseY = currentMouseY;
    updatePosition = updatePositionHandler;
    
    indexInStore = index;
  }
  
  const mouseMoveHandler = (currentMouseX, currentMouseY) => {
    if (isDraging) {
      const xDifference = currentMouseX - prevMouseX;
      const yDifference = currentMouseY - prevMouseY;
      updatePosition(elementsToRender[indexInStore].x + xDifference, elementsToRender[indexInStore].y + yDifference);
      
      prevMouseX = currentMouseX;
      prevMouseY = currentMouseY;
    }
  }
  
  const mouseupHandler = (e) => {
    isDraging = false;
  }
  
  const handleSidebarButtonClick = () => {
    elements.update(els => [...els, { x: 400, y: 400, type: 'button', id: Math.random()}]);
  }
  
  const updateElement = (elementIndex) => (x, y) => {
    elements.update((els) => update(elementIndex, {...els[elementIndex], x, y}, els));
  }
</script>

<main>
  <div class="container" on:mousemove={(e) => mouseMoveHandler(e.clientX, e.clientY)} on:mouseup={mouseupHandler}>
    {#each elementsToRender as element, index}
      {#if element.type === 'sidebar'}
        <div class="sidebar" style="top: {element.y}px; left: {element.x}px;">
          <header class="header" on:mousedown={(e) => mousedownHandler(e.clientX, e.clientY, updateElement(index), index)}>
            <h3>Sidebar</h3>
          </header>
          <div class="sidebarContainer">
            <button class="sidebarButton" on:click={handleSidebarButtonClick}>
              <div>
                Button
              </div>
            </button>
          </div>
        </div>
      {:else if element.type === 'details'}
        <div
          class="detailsContainer"
          style="top: {element.y}px; left: {element.x}px;"
          on:mousedown={(e) => mousedownHandler(e.clientX, e.clientY, updateElement(index), index)}
        >
          
        </div>
      {:else}
        <div
          class="buttonElement {activeContainer === element.id ? 'activeContainer' : ''}"
          style="top: {element.y}px; left: {element.x}px;"
          on:mousedown={(e) => mousedownHandler(e.clientX, e.clientY, updateElement(index), index, element.id)}
        ></div>
      {/if}
    {/each}
  </div>
</main>

<style>
  .container {
    width: 100vw;
    height: 100vh;
    background-color: #ffefef;
    position: relative;
  }
  
  .sidebar {
    width: 200px;
    min-height: 500px;
    max-height: calc(100vh - 100px);
    background-color: #f9f9f9;
    box-shadow: rgba(0,0,0, 0.2) 1px 1px 10px 2px;
    border-radius: 8px;
    position: fixed;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    z-index: 999999;
  }
  
  .header {
    background-color: #7777e3;
    width: 100%;
    height: 40px;
  }
  
  .header h3 {
    padding: 8px;
    margin: 0;
    color: #f9f9f9;
  }
  
  .sidebarContainer {
    display: flex;
    flex-wrap: wrap;
  }
  
  .sidebarButton {
    width: 100%;
    border-radius: 8px;
    margin: 8px;
    background-color: #9999e3;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #ffffff;
  }
  
  .sidebarButton:hover {
    background-color: #aaaaef;
    cursor: pointer;
  }
  
  .sidebarButton div {
    width: 100%;
    height: 30px;
    border: 1px solid rgba(255, 255,255, 0.6);
    border-radius: 5px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .buttonElement {
    position: fixed;
    width: 100px;
    height: 100px;
    background-color: #e40000;
    border-radius: 8px;
    box-shadow: rgba(0,0,0, 0.2) 1px 1px 10px 0px;
  }
  
  .activeContainer {
    border: 3px solid #40dfff;
  }
  
  .detailsContainer {
    width: 400px;
    height: 400px;
    background-color: #23ff23;
    border-radius: 8px;
  }
</style>