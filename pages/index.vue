<template lang="pug">
.wrapper(ref='wrapper')
  #map.map
  Modal(:basicAuth='basicAuth', :appUrl='appUrl')
  CoModal(:data='data')
</template>

<script lang="ts">
import { Component, Ref, Vue } from 'nuxt-property-decorator'
import mapboxgl from 'mapbox-gl'
import MapboxLanguage from '@mapbox/mapbox-gl-language'
import 'mapbox-gl/dist/mapbox-gl.css'

@Component({
  asyncData({ $config }) {
    const accessToken = $config.accessToken
    const basicAuth = $config.basicAuth
    const appUrl = $config.appUrl
    return { accessToken, basicAuth, appUrl }
  },
})
export default class Index extends Vue {
  @Ref() readonly wrapper!: HTMLDivElement

  map!: mapboxgl.Map
  accessToken: string = ''
  data = {}

  mounted() {
    this.map = new mapboxgl.Map({
      accessToken: this.accessToken,
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [139.7671, 35.6812],
      zoom: 9,
    })

    this.map.addControl(
      new MapboxLanguage({
        defaultLanguage: 'ja',
      }),
    )

    this.map.addControl(
      new mapboxgl.GeolocateControl({
        positionOptions: {
          enableHighAccuracy: true,
        },
        trackUserLocation: true,
        showUserHeading: true,
      }),
      'bottom-left',
    )

    this.map.addControl(new mapboxgl.NavigationControl(), 'bottom-left')

    const pngName = ['me', 'kurumaisu', 'kodomo', 'mimi']

    this.map.on('load', () => {
      pngName.forEach((name) => {
        this.map.loadImage(`/pin/${name}.png`, (error, image) => {
          if (error) throw error
          this.map.addImage(name, image!)
          this.map.addSource(name, {
            type: 'geojson',
            data: {
              type: 'FeatureCollection',
              features: [
                {
                  type: 'Feature',
                  properties: {
                    name: '山田　太郎',
                    time: '2022/09/18',
                    text: 'ゴミ箱の色分けが自分には意味がないので吹き出しの形を変えてほしいです',
                    tag: '#色覚障害,#１型色覚',
                  },
                  geometry: {
                    type: 'Point',
                    coordinates: [139.546676635742, 35.3192138671875],
                  },
                },
              ],
            },
          })

          this.map.addLayer({
            id: name,
            type: 'symbol',
            source: name,
            layout: {
              'icon-image': name,
              'icon-size': 0.5,
            },
          })

          this.map.on('click', name, (e) => {
            const description = (e.features![0] as any).properties
            if (description !== undefined) {
              this.data = description
              this.activeModal()
            }
          })
        })
      })
    })
  }

  openUrl(url: string) {
    window.open(url, '_blank')
  }

  activeModal() {
    document.body.setAttribute('data-is-active', 'true')
  }

  hiddenModal() {
    document.body.removeAttribute('data-is-active')
  }
}
</script>

<style lang="scss" scoped>
.wrap {
  position: relative;
}

.map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
}
</style>
