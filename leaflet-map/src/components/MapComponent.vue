<template>
  <div id="map" style="height: 500px; width: 100%"></div>
</template>

<script>
import L from "leaflet";

export default {
  name: "MapComponent",
  mounted() {
    // Haritayı başlat
    const map = L.map("map").setView([20, 0], 2);

    // GeoJSON dosyasını yükle
    fetch("/sade_5_test.geojson")
      .then((response) => response.json())
      .then((geojsonData) => {
        // GeoJSON'u haritaya ekle
        const geoLayer = L.geoJSON(geojsonData, {
          style: () => ({
            color: "#ff7800", // Çizgi rengi (sınır)
            weight: 2, // Çizgi kalınlığı
            fillColor: "#ffd700", // Alan rengi
            fillOpacity: 0.3, // Alan şeffaflığı
          }),
          onEachFeature: (feature, layer) => {
            // `type_en` kontrolüne göre gösterilecek isim
            const displayName =
              feature.properties.type_en === "Province"
                ? feature.properties.gn_name
                : feature.properties.admin;

            // Popup ekle
            layer.bindPopup(`<b>${displayName}</b>`);

            // Tooltip (ülke/şehir ismi) ekle
            layer.bindTooltip(displayName, {
              permanent: true, // Kalıcı
              direction: "center", // Ülkenin merkezinde
              className: "country-label", // Stil için özel sınıf
            });

            // Tooltip başlangıçta gizli olsun
            layer.getTooltip().setOpacity(0);
          },
        }).addTo(map);

        // Zoom seviyesine göre etiketleri kontrol et
        map.on("zoomend", () => {
          const currentZoom = map.getZoom();

          geoLayer.eachLayer((layer) => {
            // Belirli bir zoom seviyesinden sonra etiketleri göster
            if (currentZoom >= 5) {
              layer.getTooltip().setOpacity(1); // Göster
            } else {
              layer.getTooltip().setOpacity(0); // Gizle
            }
          });
        });
      })
      .catch((err) => console.error("GeoJSON yükleme hatası:", err));
  },
};
</script>

<style scoped>
.country-label {
  font-size: 12px;
  font-weight: bold;
  color: #333;
  text-shadow: 1px 1px 3px rgba(255, 255, 255, 0.7);
}
</style>
