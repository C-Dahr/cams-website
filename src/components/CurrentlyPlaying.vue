<template>
  <p-carousel :value="gameData" :numVisible="1" :numScroll="1" circular :autoplayInterval="3000" :showNavigators="false" :showIndicators="false" containerClass="w-full" style="width: 460px">
    <template #item="slotProps">
      <div v-if="slotProps.data.steamId">
        <a :href="getSteamPageUrl(slotProps.data)" target="_blank">
          <p-image :src="getBannerUrl(slotProps.data)" style="width: 100%;"></p-image>
        </a>
      </div>
    </template>
    <template #footer>
      <div style="height: 50px;">
        <p-card style="background:rgba(0, 0, 0, 0.60);position: relative; top: -52px;border-radius: 0px;height: 3rem"></p-card>
        <p style="position: relative; top: -116px;font-size:x-large;">
          Currently Playing
        </p>
      </div>
    </template>
  </p-carousel>
</template>
  
<script>

export default {
  name: "GameView",
  props: ['gameData'],
  data() {
    return {
    };
  },
  created() {
  },
  mounted() {

  },
  methods: {
    getBannerUrl(data) {
      var id = data.steamId;
      var banner = "https://cdn.akamai.steamstatic.com/steam/apps/"+id+"/header.jpg";
      return banner;
    },
    getSteamPageUrl(data) {
      var id = data.steamId;
      var title = data.title.replace(/ /g,"_").replace(/'/g, '');
      var steamPage = "https://store.steampowered.com/app/"+id+"/"+title+"/";
      return steamPage;
    },
  },
};

</script>
  
<style scoped>

.p-carousel-footer {
  height: 50px !important;
}

</style>