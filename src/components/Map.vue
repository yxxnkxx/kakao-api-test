<template>
  <div>
    <h2>map</h2>

    <div id="map" style="width: 500px; height: 400px"></div>
    <div>
      <form action="">
        <input type="text" v-model="search" />
        <button @click="keySearch">검색</button>
      </form>
    </div>
  </div>
</template>
<script>
export default {
  name: "KakaoMap",
  data() {
    return {
      map: null,
      search: "",
      infowindow: null,
    };
  },
  mounted() {
    window.kakao && window.kakao.maps ? this.initMap() : this.addScript();
  },

  methods: {
    initMap() {
      var container = document.getElementById("map");
      var options = {
        center: new kakao.maps.LatLng(33.450701, 126.570667),
        level: 3,
      };
      this.map = new kakao.maps.Map(container, options);

      // click event
      kakao.maps.event.addListener(this.map, "click", function (mouseEvent) {
        // 클릭한 위도, 경도 정보를 가져옵니다
        var latlng = mouseEvent.latLng;

        var message = "클릭한 위치의 위도는 " + latlng.getLat() + " 이고, ";
        message += "경도는 " + latlng.getLng() + " 입니다";

        console.log(message);

        this.map.ps = new kakao.maps.services.Places();
        this.infowindow = new kakao.maps.InfoWindow({ zIndex: 1 });
      });
    },

    addScript() {
      const script = document.createElement("script");

      script.onload = () => kakao.maps.load(this.initMap);
      script.src =
        "http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=e894c36415fadaef2e28d48b5d15ce93&libraries=services";
      document.head.appendChild(script);
    },
    keySearch() {
      this.map.ps.keywordSearch(this.search, this.placesSearchCB);
    },
    placesSearchCB(data, status) {
      this.result = data;
      if (status === kakao.maps.services.Status.OK) {
        // 검색된 장소 위치를 기준으로 지도 범위를 재설정하기위해
        // LatLngBounds 객체에 좌표를 추가합니다
        var bounds = new kakao.maps.LatLngBounds();

        for (var i = 0; i < data.length; i++) {
          this.displayMarker(data[i]);
          bounds.extend(new kakao.maps.LatLng(data[i].y, data[i].x));
        }

        // 검색된 장소 위치를 기준으로 지도 범위를 재설정합니다
        this.map.setBounds(bounds);
      }
    },
    displayMarker(place) {
      // 마커를 생성하고 지도에 표시합니다
      var marker = new kakao.maps.Marker({
        map: this.map,
        position: new kakao.maps.LatLng(place.y, place.x),
      });

      // 마커에 클릭이벤트를 등록합니다
      kakao.maps.event.addListener(marker, "click", function () {
        // 마커를 클릭하면 장소명이 인포윈도우에 표출됩니다
        this.infowindow.setContent(
          '<div style="padding:5px;font-size:12px;">' +
            place.place_name +
            "</div>"
        );
        this.infowindow.open(this.map, marker);
      });
    },
  },
};
</script>

<style scoped></style>
