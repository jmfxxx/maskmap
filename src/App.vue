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
            <select
              id="cityName"
              class="form-control"
              v-model="select.city"
              @change="updateSelect"
            >
              <!-- <option value="">請選擇縣市</option> -->
              <option
                :value="c.CityName"
                v-for="c in cityName"
                :key="c.CityName"
              >
                {{ c.CityName }}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="area" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select
              id="area"
              class="form-control"
              v-model="select.area"
              @change="updateSelect"
             
            >
              <option value="">請選擇地區</option>
              <option
                :value="a.AreaName"
                v-for="a in getAreaName(select.city)"
                :key="a.AreaName"
              >
                {{ a.AreaName }}
              </option>
            </select>
          </div>
        </div>
        <h5>
          {{ select.city }}{{ select.area }}共{{ areastore.length }}間藥局
        </h5>
        <ul>
        <li
          id="store"
          v-for="(item, index) in areastore"
          :key="index"
          @click="getPosition(index)"
        >
          <h6>店家名稱:{{ item.properties.name }}</h6>
          <p>電話:{{ item.properties.phone }}</p>
          <p>地址:{{ item.properties.address }}</p>
          <span style="color:purple"
            >大人:{{ item.properties.mask_adult }}</span
          >
         
          <span style="color:blue">    小孩:{{ item.properties.mask_child }}</span><br>
          
        </li>
        </ul>
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
import L from "leaflet";
let osm = {};

export default {
  name: "App",
  components: {},
  data: function() {
    return {
      data: [],
      cityName,
      select: {
        city: "臺北市",
        area: "",
      },
      areastore: [],
      getName: "", //取得店名
    };
  },
  methods: {
    getAreaName(getcity) {
      console.log(getcity);
      return this.cityName.find((city) => city.CityName === getcity).AreaList;
    },
    // 更新地圖
    updateMap() {
      // 清除marker
      this.removeMarker();
      // 判斷是有選擇區域
      const stores = this.data.filter((pharmacy) => {
        //  如果區域沒選
        if (!this.select.area) {
          let a = this.select.city;
          return pharmacy.properties.address.match(a);
        }
        // 區域有選擇
        let b = this.select.city+this.select.area ;
        return pharmacy.properties.address.match(b);
      });

// 複制一份內容深烤貝，給this.areastore
      this.areastore = Object.assign([], stores);
      // 畫制圖程坐標
      stores.forEach((pharmacy) => {
        // 透過藥局經緯度疊加標記
        L.marker([
          pharmacy.geometry.coordinates[1],
          pharmacy.geometry.coordinates[0],
        ]).addTo(osm).bindTooltip(`<p><strong style="font-size: 20px;">${
          pharmacy.properties.name
        }</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 : ${
            pharmacy.properties.mask_adult
              ? `${pharmacy.properties.mask_adult} 個`
              : "未取得資料"
          } / 兒童 : ${
          pharmacy.properties.mask_child
            ? `${pharmacy.properties.mask_child} 個`
            : "未取得資料"
        }</strong><br>
          地址: ${pharmacy.properties.address}<br>
          電話: ${pharmacy.properties.phone}<br>
          <small>最後更新時間: ${pharmacy.properties.updated}</small>`);
      });
      let newy = this.areastore[0].geometry.coordinates[0];
      let newx = this.areastore[0].geometry.coordinates[1];

      // osm.panTo(new L.LatLng(newx, newy));
     osm.setView([newx,newy],10)
    },

    removeMarker() {
      // 清除圖層
      // clear markers
      osm.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          osm.removeLayer(layer);
        }
      });
    },

    updateSelect() {
      this.removeMarker();
      this.updateMap();
      //  const store = this.data.find(item => item.properties.address.match(this.select.area));
    },
    // 到指定藥局
    getPosition(index) {
      // 移到最上面
      window.scrollTo(0, 0);
      // console.log(index);
      console.log(this.areastore[index]);
      let y = this.areastore[index].geometry.coordinates[0];
      let x = this.areastore[index].geometry.coordinates[1];
      let marker1 = L.marker(
        [x, y],{
          title: this.areastore[index].properties.name,
        }
      ).addTo(osm);

      marker1
        .bindPopup(
          `<p><strong style="font-size: 20px;">${
            this.areastore[index].properties.name
          }</strong></p>
          <strong style="font-size: 16px; color: #d45345;">口罩剩餘：成人 : ${
            this.areastore[index].properties.mask_adult
              ? `${this.areastore[index].properties.mask_adult} 個`
              : "未取得資料"
          } / 兒童 : ${
            this.areastore[index].properties.mask_child
              ? `${this.areastore[index].properties.mask_child} 個`
              : "未取得資料"
          }</strong><br>
          地址: ${this.areastore[index].properties.address}<br>
          電話: ${this.areastore[index].properties.phone}<br>
          <a href="https://www.google.com.tw/maps/place/${this.areastore[index].properties.address}" target="_blank"> Google地圖</a><br>
          <small>最後更新時間: ${
            this.areastore[index].properties.updated
          }</small>`
        )
        .openPopup();
      // osm.panTo(new L.LatLng(x, y));
      osm.setView([x,y],15)
    },
  },
  computed: {},
  watch: {},
  mounted() {
    // leaflet 產生地圖物件，並指定在那個#ID上，顯示設定位置(中心點center坐標)指定縮放比例為18
    osm = L.map("map", { center: [25.042474, 121.513729], zoom: 18 });
    // titleLayer主要是針對地圖上操作進行設定

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 20,
    }).addTo(osm);

    const api =
      "https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json";
    this.$http.get(api).then((res) => {
      console.log(res);
      this.data = res.data.features;
      this.updateMap();
    });
  },
};
console.log("aaa");
console.log(osm);
</script>

<style lang="scss">
@import "bootstrap/scss/bootstrap";

#map {
  position: relative;
  height: 100vh;
  
}

#store {
  background: rgb(235, 208, 233);
  border-bottom: 2px solid rgb(54, 53, 53);
}
#store p {
  margin: 0;
  padding: 0;
  font-size: 14px;
}
#store:hover {
  background: yellow;
}
a {
    cursor: pointer;
  }
  ul{
    padding:0
  }
  li{
    list-style: none;
  }
</style>
