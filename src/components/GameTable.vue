<template>
    <p-datatable class="gamesTable shadow-8" v-model:filters="filters" :value="gameData" paginator :rows="10" :rowsPerPageOptions="[5, 10, 20, 50]" dataKey="title"
            filterDisplay="menu" :loading="loading" removableSort :globalFilterFields="['title','genre']" v-model:expandedRows="expandedRows" tableStyle="width: 50rem;height: 50rem">
        <template #header>
            <div class="flex justify-content-between">
                <div class="flex flex-row">
                  <p-button class="mr-2" type="button" icon="pi pi-refresh" label="Refresh" outlined @click="refresh()" />
                  <p-button class="mr-2" type="button" icon="pi pi-filter-slash" label="Clear" outlined @click="clearFilter()" />
                </div>
                <div class="flex flex-row">
                  <p-iconField iconPosition="left">
                      <p-inputIcon>
                          <i class="pi pi-search" />
                      </p-inputIcon>
                      <p-inputText v-model="filters['global'].value" placeholder="Keyword Search" />
                  </p-iconField>
                </div>
            </div>
        </template>
        <template #empty>
            <p class="flex flex-column align-items-center justify-content-center">No games found.</p>
        </template>
        <template #loading> Loading game data. Please wait. </template>
        <p-column expander style="width: 5rem" />
        <p-column field="title" header="Title" sortable style="min-width: 18rem">
            <template #body="{ data }">
                <div class="flex flex-row align-items-center">
                    <div class="mr-3">
                        <p-image :src="getIconUrl(data)" width="32"></p-image>
                    </div>
                    <div>
                        {{ data.title }}
                    </div>
                </div>
            </template>
        </p-column>
        <p-column field="completion" header="Completion" filterField="completion" :showFilterMatchModes="false" :filterMenuStyle="{ width: '14rem' }" sortable style="min-width: 14rem">
            <template #body="{ data }">
                <p-tag :value="data.completion" :severity="getSeverity(data.completion)" />
            </template>
            <template #filter="{ filterModel }">
                <p-multiSelect class="p-column-filter filter-popup" v-model="filterModel.value" :options="statuses" placeholder="Any" :showToggleAll="false">
                    <template #option="slotProps">
                        <div class="flex align-items-center gap-2 filter-popup">
                            <p-tag :value="slotProps.option" :severity="getSeverity(slotProps.option)" />
                        </div>
                    </template>
                </p-multiSelect>
            </template>
        </p-column>
        <p-column field="hours" header="Time Played" sortable style="min-width: 12rem">
            <template #body="{ data }">
                {{ data.hours }} hours
            </template>
        </p-column>
        <p-column field="rating" header="Rating" filterField="rating" :showFilterMatchModes="false" :filterMenuStyle="{ width: '14rem' }" sortable style="min-width: 14rem">
            <template #body="{ data }">
                <p-button v-if=data.rating severity="info" text link v-tooltip.bottom="{ value: 'View Metacritic' }">
                  <a :href="getMetacriticPageUrl(data)" target="_blank">
                    <font-awesome-icon v-for="index in getRatingStars(data.rating)" :key="index" class="text-xl" icon="fa-solid fa-star" :color="getStarColor(data.rating)" />
                    <font-awesome-icon v-if="getRatingHalfStar(data.rating)" class="text-xl" icon="fa-solid fa-star-half-stroke" :color="getStarColor(data.rating)" />
                    <font-awesome-icon v-for="index in getRatingStars(String(10-Number(data.rating)))" :key="index" class="text-xl" icon="fa-regular fa-star" :color="getStarColor(data.rating)" />
                    <!--
                    <p-tag :value="data.rating" :severity="getRatingColor(data.rating)"/>
                    -->
                  </a>
                </p-button>
            </template>
            <template #filter="{ filterModel }">
                <p-multiSelect v-model="filterModel.value" :options="ratings" placeholder="Any" class="p-column-filter filter-popup" :showToggleAll="false">
                    <template #option="slotProps">
                        <div class="flex align-items-center gap-2 filter-popup">
                          <p-tag :value="slotProps.option" :severity="getRatingColor(slotProps.option)"/>
                        </div>
                    </template>
                </p-multiSelect>
            </template>
        </p-column>
        <template #expansion="slotProps">
            <div class="flex flex-row p-3">
                <div>
                    <p-image :src="getBannerUrl(slotProps.data)" width="460"></p-image>
                </div>
                <div class="flex flex-column align-content-center pl-5 pr-5 w-full h-full">
                  <div class="flex flex-row justify-content-between align-items-center text-xl font-medium text-900 w-full">
                    <span>{{ slotProps.data.title }}</span>
                    <p-button v-if="slotProps.data.steamId" severity="info" text link aria-label="Steam" v-tooltip.bottom="{ value: 'Steam Page' }">
                      <template #icon>
                        <a :href="getSteamPageUrl(slotProps.data)" target="_blank">
                          <font-awesome-icon class="text-2xl" icon="fa-brands fa-steam" color="grey" />
                        </a>
                      </template>
                    </p-button>
                  </div>
                  <div v-if="slotProps.data.date" class="font-medium text-secondary text-sm mt-2">
                    <span><i>Finished {{ slotProps.data.date }}</i></span>
                  </div>
                  <div v-if="!slotProps.data.date" class="font-medium text-secondary text-sm mt-2">
                    <span><i>DNF</i></span>
                  </div>
                  <div v-if="slotProps.data.genre" class="mt-3">
                    <p-tag v-for="item in formatTags(slotProps.data.genre)" :value=item severity="secondary" v-bind:key="item" class="mr-2"/>
                  </div>
                  <div v-if="slotProps.data.notes" class="font-medium text-secondary text-sm mt-3">
                    <span>{{ truncateString(slotProps.data.notes, 250) }}</span>
                    <span v-if="slotProps.data.notes.length > 250">
                      <span>...</span>
                      <p-button class="p-0" link><span @click="showReviewDialog(slotProps.data)">[Show More]</span></p-button>
                    </span>
                  </div>
                </div>
            </div>
        </template>
        <p-dialog v-model:visible="showReviewDialogVal" modal :style="{ width: '25rem' }" dismissableMask>
              <template #header>
                <div class="inline-flex align-items-center justify-content-center gap-2">
                  <div v-if="reviewIcon" class="mr-3">
                    <p-image :src="getIconUrl(reviewIcon)" width="32"></p-image>
                  </div>
                  <span class="text-xl">
                      {{ reviewTitle }}
                  </span>
                </div>
              </template>
              <span>{{ review }}</span>
            </p-dialog>
    </p-datatable>
</template>
  
<script>
  import { FilterMatchMode, FilterOperator } from '@primevue/core/api'
  
  export default {
    name: "GameView",
    props: ['gameData', 'loading'],
    data() {
      return {
        filters: null,
        statuses: ['Backlog', 'Finished', '100%', 'Abandoned', 'In Progress'],
        ratings: ['10', '9', '8', '7', '6', '5', '4', '3', '2', '1'],
        expandedRows: [],
        scoreType: 'false',
        review: null,
        reviewTitle: null,
        reviewIcon: null,
        showReviewDialogVal: false,
      };
    },
    created() {
      this.initFilters();
    },
    methods: {
      formatTags(value) {
         return value.split(",");
      },
      formatCurrency(value) {
        return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
      },
      clearFilter() {
        this.initFilters();
      },
      initFilters() {
        this.filters = {
          global: { value: null, matchMode: FilterMatchMode.CONTAINS },
          title: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
          completion: { value: ['Finished', '100%', 'Abandoned'], matchMode: FilterMatchMode.IN },
          rating: { value: null, matchMode: FilterMatchMode.IN },
        };
      },
      getSeverity(status) {
        switch (status) {
          
          case 'Abandoned':
            return 'danger';
          
          case 'Finished':
            return 'success';
          
          case 'Backlog':
            return 'contrast';
          
          case 'In Progress':
            return 'warn';
          
          case '100%':
            return 'info';
        }
      },
      getStarColor(rating) {
        switch (rating) {
          
          case '10':
          case '9':
            return '#56daf5';
          
          case '8':
          case '7':  
            return '#5ff56c';
          
          case '6':
          case '5':
            return '#e8f556';
          
          case '4':
          case '3':
            return '#f5b856';
  
          case '2':
          case '1':
            return '#f55656';

        }
      },
      getRatingColor(rating) {
        switch (rating) {
          
          case '10':
          case '9':
            return 'info';
          
          case '8':
          case '7':  
            return 'success';
          
          case '6':
          case '5':
            return 'primary';
          
          case '4':
          case '3':
            return 'warn';
  
          case '2':
          case '1':
            return 'danger';

        }
      },
      getRatingStars(rating) {
        switch (rating) {
          
          case '10':
            return 5;
          
          case '9':
          case '8':  
            return 4;
          
          case '7':
          case '6':
            return 3;
          
          case '5':
          case '4':
            return 2;
  
          case '3':
          case '2':
            return 1;
          
          case '1':
            return 0;

        }
      },
      getRatingHalfStar(rating) {
        switch (rating) {
          
          case '9':
          case '7':
          case '5':
          case '3':
          case '1':
            return true;
          
          default:
            return false;

        }
      },
      getEmptyStars(rating) {
        switch (rating) {
          
          case '10':
          case '9':
            return 0;
          
          case '8':
          case '7':  
            return 1;
          
          case '6':
          case '5':
            return 2;
          
          case '4':
          case '3':
            return 3;
  
          case '2':
          case '1':
            return 4;

        }
      },
      getIconUrl(data) {
        var icon;
        if(data.steamIcon != null && data.steamIcon != ""){
          var id = data.steamId;
          var hash = data.steamIcon;
          icon = "http://media.steampowered.com/steamcommunity/public/images/apps/"+id+"/"+hash+".jpg";
        }else{
          var title = data.title.toLowerCase().replace(/ /g,"_").replace(/'/g, '');
          icon = "/game_assets/icons/"+title+"_icon.png";
        }
        return icon;
      },
      getBannerUrl(data) {
        var banner;
        if(data.steamId != null && data.steamId != ""){
          var id = data.steamId;
          banner = "https://cdn.akamai.steamstatic.com/steam/apps/"+id+"/header.jpg";
        }else{
          var title = data.title.toLowerCase().replace(/ /g,"_").replace(/'/g, '');
          banner = "/game_assets/banners/"+title+"_banner.png";
        }
        return banner;
      },
      refresh() {
        this.$emit('refresh-data');
      },
      getSteamPageUrl(data) {
        var id = data.steamId;
        var title = data.title.replace(/ /g,"_").replace(/'/g, '');
        var steamPage = "https://store.steampowered.com/app/"+id+"/"+title+"/";
        return steamPage;
      },
      getMetacriticPageUrl(data) {
        var title = data.title.replace(/— /g,"").replace(/- /g,"").replace(/ /g,"-").replace(/'/g, '').replace(/:/g, '').replace(/™/g, '').toLowerCase();
        var metacriticPage = "https://www.metacritic.com/game/"+title+"/";
        return metacriticPage;
      },
      truncateString(yourString, maxLength) {
        const index = yourString.indexOf(" ", maxLength);
        return index === -1 ? yourString : yourString.substring(0, index)
      },
      showReviewDialog(gameData) {
        this.review = null;
        this.reviewTitle = null;
        this.reviewIcon = null;
        this.review = gameData.notes;
        this.reviewTitle = gameData.title;
        this.reviewIcon = gameData;
        if(gameData.title != ""){
          this.showReviewDialogVal = true;
        }
      },
    },
  };
</script>
  
<style scoped>
  
  .gamesTable {
    font-family: Avenir, Helvetica, Arial, sans-serif !important;
  }

  .filter-popup {
    font-family: Avenir, Helvetica, Arial, sans-serif !important;
  }

  .p-button-label {
    font-family: Avenir, Helvetica, Arial, sans-serif !important;
  }
  
</style>