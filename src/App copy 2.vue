<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇所在區域 -->
      <div class="toolbox col-sm-3 p-2 bg-info">
        <div class="form-group d-flex">
          <label for="cityName" class="col-form-label mr-2 text-right"
            >縣市</label
          >
          <div class="flex-fill">

  <select id="cityName" class="form-control" v-model="select.city">
    <option value="">請選擇縣市</option>
    <option :value="c.CityName" v-for="c in cityName" :key="c.CityName" @change="updateMap()">
      {{ c.CityName }}
    </option>
  </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="area" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="area" class="form-control" v-model="select.area">
               <option value="">請選擇地區</option>
                  <option :value="a.AreaName" v-for="a in getAreaName(select.city)" :key="a.AreaName" @change="updateMap()">
      {{ a.AreaName }}
    </option>

            </select>
  
          </div>
        </div>
      </div>

      <!-- 顯示藥局位置 -->
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import cityName from "./assets/cityName.json";
import L from 'leaflet';
let openStreetMap = {};





export default {
  name: "App",
  components: {},
  data: function() {
    return {
      cityName,
      select: {     
      city: '臺北市',
      area: '中正區'},
      data:[],
  
  
    };
  },
  methods:{
       getAreaName(getcity){
      //  console.log(getcity)
         return this.cityName.find(city=>city.CityName===getcity).AreaList;
     
      },


      removeMarker(){
      // 清除圖層
          // clear markers
    openStreetMap.eachLayer((layer) => {
      if (layer instanceof L.Marker) {
        openStreetMap.removeLayer(layer);
      }
    });
  
      },
    updateMap() { 
        const  pharmacies = this.data.filter((pharmacy) => {
        if (!this.select.area) {
       return pharmacy.properties.county === this.select.city;
        }
         return pharmacy.properties.town === this.select.area;
      });
    // 畫制圖程坐標
    console.log(pharmacies)
     pharmacies.forEach((pharmacy) => {
        // 透過藥局經緯度疊加標記  
      
        L.marker([
          pharmacy.geometry.coordinates[1],
          pharmacy.geometry.coordinates[0],
        ]).addTo(openStreetMap).bindTooltip(`<p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 : ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 : ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}</strong><br>
          地址: ${pharmacy.properties.address}<br>
          電話: ${pharmacy.properties.phone}<br>
          <small>最後更新時間: ${pharmacy.properties.updated}</small>`);
          // bindPopup

          // 24.1369609,120.654491
      });
    },
   
    },
    computed: {

    },
    mounted(){
// leaflet 產生地圖物件，並指定在那個#ID上，顯示設定位置(中心點center坐標)指定縮放比例為18
     openStreetMap = L.map('map',{center:[25.042474,121.513729],zoom:18,});
// titleLayer主要是針對地圖上操作進行設定
 
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom:20,
    }).addTo(openStreetMap);

      const api = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json';
      this.$http.get(api).then((response)=>{
        console.log(response)
        this.data=response.data.features;
        updateMap();
      })


  },
    
    }
    console.log('1111')
  console.log(openStreetMap)
  
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";
 
#map {
  position: relative;
  height: 100vh;
}
</style>
