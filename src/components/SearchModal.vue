<template>
  <transition name="modal" appear>
    <div class="modal-overlay">
      <!------- handler pop -------->
      <div class="mid-pop pop-wrap">
        <div class="pop-header">
          <h3 class="h3-tit">검색</h3>
          <slot name="modalHeader"></slot>
        </div>
        <div class="pop-container" :class="{ 'overflow-y': true }">
          <slot name="modalContainer"></slot>
        </div>
      </div>
    </div>

    <!------- handler pop -------->
  </transition>
</template>

<script setup lang="ts"></script>
<style>
.mid-pop {
  width: 600px !important;
}
/***   popup   ***/
.pop-wrap {
  z-index: 1090;
  border: 1px #ddd solid;
  border-radius: 4px;
  background: #fff;
  padding: 20px 20px;
  margin: 0 auto;
  box-shadow: 1px 1px 5px #ccc;
}

.modal-overlay {
  align-items: center;
  justify-content: center;
  position: fixed;
  z-index: 200;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  overflow-y: auto;
  scroll-padding-top: 2rem;
}
.pop-header {
  display: flex;
  justify-content: space-between;
  position: relative;
}
</style>
