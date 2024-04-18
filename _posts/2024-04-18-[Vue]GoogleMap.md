---
title:  "Google Map API"
excerpt: "[Vue] Using Google Map API "

categories:
  - Blog
tags:
  - [Blog, jekyll, Github]
last_modified_at: 2024-03-10


---

#### 0. 들어가면서

- 현재 프로젝트를 진행하면서 몇 분 간격으로 기기의 위치를 지도 상에 나타나게 해야 했습니다. 지도는 Google Map을 사용하였고 현재 위치는 Google Map Marker를 통해서 나타냈습니다. Google Map Api를 이용하면서 Api 이용 과정과 실제 코드에서 사용하는 것을 기록하고자 합니다.


---

### 1. Procedure & Restriction

- Google Map Platform에서 API KEY를 발급 받습니다.
  - Link : https://cloud.google.com/maps-platform/
- Keys & Credentials에서 해당 API KEY에 접근성을 제한할 수 있습니다 
![image info](/assets/img/googleMap.png)
<img src="/assets/img/googleMap.png" alt="" width="0" height="0">

<br />

### 2. Application Code

- Google 계정으로 발급 받은 GOOGLE MAP API KEY가 필요합니다
  
  
<br/>

index.html
```html
<script src="https://maps.googleapis.com/maps/api/js?key= YOUR_GOOGLE_MAP_API_KEY "></script>
```
- index.html에 script 파일 추가

<br/>

map.vue
```javascript
<template>
  <div style="height:300px;" id="map"></div> 
</template>

<script>
export default {
  name: "GoogleMap",
  props :{
    data : Object
  },
  watch: {
    data : {
      handler(val){
        this.mapCenter.lat = Number(val.latitude);
        this.mapCenter.lng = Number(val.longitude);
        this.initMap();
      },
    }
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = new google.maps.Map(document.getElementById("map"), 
        center: this.mapCenter,//최초 지도가 보여지는 위치 = Marker 위치
        zoom: 20, //zoom size 지정
        maxZoom: 20, //최대 zoom size 지정
        minZoom: 3, //최소 zoom size 지정
        streetViewControl: true,
        mapTypeControl: true,
        fuulscreenControl: true,
        zoomControl: true,
      });
        this.setMarker(this.mapCenter, "H")
    },
    setMarker(Points, Label) {
      const markers = new google.maps.Marker({
        position: Points,
        map: this.map,
        label: {
          text: Label,
          color: "#FFF",
        },
      });
    },
  },
  data() {
    return {
      map: null,
      mapCenter: { lat: 37, lng: 127 }
    };
  },
};
</script>
```
- initMap() : Google Map Option 설정 및 지도를 보여줍니다
- setMarker() : Google Map상에 Marker를 보여줍니다
- Parnet Component에서 받아오는 data 값의 변화에 따라 지도에 표기되는 Marker 위치 변경합니다


---

### Refernce 
- https://wp.swing2app.co.kr/knowledgebase/googlemap-apikey/
- https://developers.google.com/maps/documentation/javascript/get-api-key?hl=ko