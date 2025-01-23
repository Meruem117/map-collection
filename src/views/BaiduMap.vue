<template>
    <div class="page">
        <div class="map-box">
            <div class="map" :id="id"></div>
        </div>
        <div class="operate-box">
            <v-btn variant="tonal" @click="startPath">Start</v-btn>
            <v-btn variant="tonal" @click="pausePath">Pause</v-btn>
            <v-btn variant="tonal" @click="resumePath">Resume</v-btn>
            <v-btn variant="tonal" @click="stopPath">Stop</v-btn>
        </div>
    </div>
</template>

<script>
/* eslint-disable no-undef */
export default {
    name: 'BaiduMap',
    data() {
        return {
            id: 'b-map',
            map: null,
            city: {
                name: '常州市',
                zoom: 11,
                center: new BMapGL.Point(119.64489, 31.629129)
            },
            path: [{
                'lng': 116.297611,
                'lat': 40.047363
            }, {
                'lng': 116.302839,
                'lat': 40.048219
            }, {
                'lng': 116.308301,
                'lat': 40.050566
            }, {
                'lng': 116.305732,
                'lat': 40.054957
            }, {
                'lng': 116.304754,
                'lat': 40.057953
            }, {
                'lng': 116.306487,
                'lat': 40.058312
            }, {
                'lng': 116.307223,
                'lat': 40.056379
            }],
            type: 'track',
            // TrackAnimation
            trackAnimation: null,
            // LuShu
            lushu: null,
            // Track
            track: null,
            trackData: [],
            trackLine: null,
            trackPoint: null
        }
    },
    mounted() {
        this.initMap()
    },
    methods: {
        initMap() {
            this.map = new BMapGL.Map(this.id)
            this.map.centerAndZoom(this.city.center, this.city.zoom)
            this.map.enableScrollWheelZoom(true)
            // this.map.setMapStyleV2({
            //     styleId: '073b242e4be0bc629bf89b95f6870c3c'
            // })
            this.initPolygon(this.city.name)
            this.initMarker(this.city.center)
            // this.addTrackAnimation()
            // this.addLuShu()
            this.addTrack()
        },

        initPolygon(city) {
            let that = this
            let boundary = new BMapGL.Boundary()
            boundary.get(city, function (res) {
                res.boundaries.forEach(item => {
                    let points = item.split(';')
                    let bPoints = points.map(item2 => {
                        let loc = item2.split(',')
                        return new BMapGL.Point(parseFloat(loc[0]), parseFloat(loc[1]))
                    })
                    let polygon = new BMapGL.Polygon(bPoints, {
                        fillColor: '#e0f2fe',
                        fillOpacity: 0.5,
                        strokeColor: '#0ea5e9',
                        strokeWeight: 2,
                        strokeOpacity: 0.5
                    })
                    that.map.addOverlay(polygon)
                })
            })
        },

        initMarker(location) {
            let marker = new BMapGL.Marker(location)
            this.map.addOverlay(marker)
        },

        // TrackAnimation
        addTrackAnimation() {
            this.map.centerAndZoom(new BMapGL.Point(116.297611, 40.047363), 17)
            const points = this.path.map(item => {
                return new BMapGL.Point(item.lng, item.lat)
            })
            const line = new BMapGL.Polyline(points)
            this.trackAnimation = new BMapGLLib.TrackAnimation(this.map, line, {
                overallView: true,
                tilt: 30,
                duration: 20000,
                delay: 300
            })
            setTimeout(() => {
                this.trackAnimation.start()
            }, 3000)
        },

        // LuShu
        addLuShu() {
            this.map.centerAndZoom(new BMapGL.Point(116.297611, 40.047363), 17)
            const points = this.path.map(item => {
                return new BMapGL.Point(item.lng, item.lat)
            })
            let polyline = new BMapGL.Polyline(points, {
                clip: false,
                geodesic: true,
                strokeWeight: 3
            })
            this.lushu = new BMapGLLib.LuShu(this.map, points, {
                geodesic: true,
                autoCenter: true,
                icon: new BMapGL.Icon(require('../assets/logo.svg'), new BMapGL.Size(35, 35)),
                enableRotation: true
            })
            this.map.addOverlay(polyline)
        },

        startLuShu() {
            if (this.lushu) {
                this.lushu.start()
            }
        },

        pauseLuShu() {
            if (this.lushu) {
                this.lushu.pause()
            }
        },

        stopLuShu() {
            if (this.lushu) {
                this.lushu.stop()
            }
        },
        // Track
        addTrack() {
            this.track = new Track.View(this.map, {
                lineLayerOptions: {
                    style: {
                        strokeWeight: 8,
                        strokeLineJoin: 'round',
                        strokeLineCap: 'round'
                    }
                }
            })
            this.trackData = this.path.map(item => {
                let point = new BMapGL.Point(item.lng, item.lat)
                let trackPoint = new Track.TrackPoint(point)
                return trackPoint
            })
            this.trackLine = new Track.LocalTrack({
                trackPath: this.trackData,
                duration: 60,
                style: {
                    sequence: true,
                    marginLength: 32,
                    arrowColor: '#fff',
                    strokeTextureUrl: '//mapopen-pub-jsapi.bj.bcebos.com/jsapiGlgeo/img/down.png',
                    strokeTextureWidth: 64,
                    strokeTextureHeight: 32,
                    traceColor: [27, 142, 236]
                }
            })
            this.track.addTrackLine(this.trackLine)
            this.track.focusTrack(this.trackLine)
            this.trackPoint = new Track.GroundPoint({
                point: this.trackData[0].getPoint(),
                style: {
                    url: '//mapopen-pub-jsapi.bj.bcebos.com/jsapiGlgeo/img/car.png',
                    level: 18,
                    scale: 1,
                    size: new BMapGL.Size(16, 32),
                    anchor: new BMapGL.Size(0.5, 0.5),
                }
            })
            this.trackLine.setMovePoint(this.trackPoint)
        },

        startTrack() {
            if (this.trackLine) {
                this.trackLine.startAnimation()
            }
        },

        pauseTrack() {
            if (this.trackLine) {
                this.trackLine.pauseAnimation()
            }
        },

        resumeTrack() {
            if (this.trackLine) {
                this.trackLine.resumeAnimation()
            }
        },

        stopTrack() {
            if (this.trackLine) {
                this.trackLine.stopAnimation()
            }
        },

        startPath() {
            if (this.type == 'lushu') {
                this.startLuShu()
            } else if (this.type == 'track') {
                this.startTrack()
            }
        },

        pausePath() {
            if (this.type == 'lushu') {
                this.pauseLuShu()
            } else if (this.type == 'track') {
                this.pauseTrack()
            }
        },

        resumePath() {
            if (this.type == 'track') {
                this.resumeTrack()
            }
        },

        stopPath() {
            if (this.type == 'lushu') {
                this.stopLuShu()
            } else if (this.type == 'track') {
                this.stopTrack()
            }
        },
    }
}
</script>

<style lang="less" scoped>
.page {
    position: relative;
    width: 100%;
    height: 100%;

    .map-box {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;

        .map {
            width: 100%;
            height: 100%;
        }
    }

    .operate-box {
        position: absolute;
        top: 30px;
        left: 30px;
        z-index: 1000;
    }
}
</style>
