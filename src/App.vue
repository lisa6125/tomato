<template lang="pug">
  #app
    //- 避免元件在換頁時被銷毀
    keep-alive
      router-view
</template>
