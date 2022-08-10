<template>
  <vs-card style="width: 150px">
    <template #title>
      <p class="card-title">Здесь могла бы быть моя фотография</p>
    </template>
    <template #img>
      <img style="width: 150px; height: 100px; object-fit: scale-down" src="https://vseraskraski.com/images/8/8/2/kaktus.png" alt="">
<!--      <div class="background-plant"></div>-->
    </template>
    <template #text>
      <p class="card-subtitle">
        но пока кактус
      </p>
    </template>
  </vs-card>
</template>

<script>
export default {
  data() {
    return {}
  }
}
</script>

<style scoped>
/*.background-plant {*/
/*  background-image: url('https://w7.pngwing.com/pngs/438/340/png-transparent-cactaceae-drawing-smile-art-family-smile-people-grass-cactus.png');*/
/*  background-repeat: no-repeat;*/
/*  background-position: center;*/
/*  background-size: cover;*/
/*  width: 150px;*/
/*  height: 100px;*/
/*}*/
.card-title {
  font-size: 12px;
  font-style: italic;
  font-weight: 600;
}

.card-subtitle {
  font-size: 10px;
  font-style: italic;
  font-weight: 400;
}
</style>
