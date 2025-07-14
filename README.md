<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>수성구 건강권 점수 지도</title>
  <style>
    #map { width: 100%; height: 500px; margin-bottom: 20px; }
    #scoreDisplay { font-size: 20px; font-weight: bold; }
  </style>
  <script type="text/javascript" src="//dapi.kakao.com/v2/maps/sdk.js?appkey=YOUR_APP_KEY&libraries=services"></script>
</head>
<body>
  <h2>내 위치 기반 수성구 건강 인프라 점수 확인</h2>
  <div>
    <input type="text" id="addressInput" placeholder="주소를 입력하세요 (예: 수성구 수성로 194)">
    <button onclick="searchAddress()">확인</button>
  </div>
  <div id="map"></div>
  <div id="scoreDisplay"></div>

  <script>
    var mapContainer = document.getElementById('map');
    var mapOption = {
      center: new kakao.maps.LatLng(35.8592, 128.6306),
      level: 5
    };

    var map = new kakao.maps.Map(mapContainer, mapOption);
    var geocoder = new kakao.maps.services.Geocoder();

    // 테스트용 인프라 데이터 (주소 없이 좌표 기반)
    var infraPoints = [
      { name: "수성구보건소", lat: 35.8583, lng: 128.6295, type: "보건소" },
      { name: "천주성삼병원", lat: 35.8415, lng: 128.7072, type: "공공병원" },
      { name: "범어공원", lat: 35.8509, lng: 128.6374, type: "공원" },
      { name: "수성국민체육센터", lat: 35.8430, lng: 128.6231, type: "체육시설" }
    ];

    // 마커 미리 지도에 표시
    infraPoints.forEach(function(p) {
      var marker = new kakao.maps.Marker({
        map: map,
        position: new kakao.maps.LatLng(p.lat, p.lng),
        title: p.name
      });
    });

    function searchAddress() {
      var addr = document.getElementById("addressInput").value;
      geocoder.addressSearch(addr, function(result, status) {
        if (status === kakao.maps.services.Status.OK) {
          var coords = new kakao.maps.LatLng(result[0].y, result[0].x);
          map.setCenter(coords);
          calculateScore(coords);
        } else {
          alert("주소를 찾을 수 없습니다");
        }
      });
    }

    function calculateScore(centerCoords) {
      let score = 0;
      infraPoints.forEach(p => {
        var dist = getDistance(centerCoords.getLat(), centerCoords.getLng(), p.lat, p.lng);
        if (dist <= 1.0) score += 20;  // 1km 내 인프라마다 20점 부여
      });
      document.getElementById("scoreDisplay").innerText = "건강권 점수: " + score + "점 (100점 만점)";
    }

    // Haversine 거리 계산 (단위: km)
    function getDistance(lat1, lon1, lat2, lon2) {
      function toRad(x) { return x * Math.PI / 180; }
      var R = 6371; // Earth radius (km)
      var dLat = toRad(lat2 - lat1);
      var dLon = toRad(lon2 - lon1);
      var a = Math.sin(dLat/2) * Math.sin(dLat/2) +
              Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
              Math.sin(dLon/2) * Math.sin(dLon/2);
      var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
      return R * c;
    }
  </script>
</body>
</html>
