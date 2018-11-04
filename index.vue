<template>
    <div class="bdmap">
        <div id="bdmap-container"></div>
    </div>
</template>
<script>
// import {Toast} from 'mint-ui'
import {mapTempleList} from '../../service/getData'
import MarkerIcon from '../../images/marker-x.png'
export default {
    data(){
        return {
            map:null
        }
    },
    mounted(){
        let vm = this 
        let timer = setInterval(()=>{
            // 百度地图组件已从index.html文件引入
            if(BMap){
                clearInterval(timer)
                vm.initMap()
            }
        })
    },
    methods:{
        // 初始化地图
        initMap(){
            let vm = this
            vm.$nextTick(()=>{
                let map = new BMap.Map('bdmap-container')
                let initMapCenter = new BMap.Point(116.404, 39.915);    
                map.centerAndZoom(initMapCenter, 10);                  
                map.enableScrollWheelZoom(true);                        
                // 添加带有定位的导航控件
                let navigationControl = new BMap.NavigationControl({
                    // 靠左上角位置
                    anchor: BMAP_ANCHOR_BOTTOM_LEFT,
                    // LARGE类型
                    type: BMAP_NAVIGATION_CONTROL_LARGE,
                    // 启用显示定位
                    enableGeolocation: true
                });
                map.addControl(navigationControl);
                // 添加定位控件
                let geolocationControl = new BMap.GeolocationControl({anchor: BMAP_ANCHOR_BOTTOM_RIGHT});
                map.addControl(geolocationControl);
                vm.map = map
                vm.loadTempleList()
            })
        },
        loadTempleList(){
            let vm = this
            mapTempleList().then(ret=>{
                if(ret.ret==1){
                    vm.addMarker(ret.data)
                }else{
                    Toast(ret.info);
                }
            })
        },
        addMarker(data,animate){
            let vm = this
            let map = vm.map
            let pointArray = [];
            for(let i=0,len=data.length;i<len;i++){
                let curr = data[i];
                if(curr.jingweidu){
                    let point = new BMap.Point(curr.jingweidu.split(',')[0],curr.jingweidu.split(',')[1]);
                    let icon = new BMap.Icon(MarkerIcon, new BMap.Size(25, 25), {
                        offset: new BMap.Size(30, 30), // 指定定位位置
                        // imageOffset: new BMap.Size(0,-110) ,// 设置图片偏移
                        imageSize:new BMap.Size(30, 25)
                    });
                    let marker = new BMap.Marker(point,{icon}); 
                    map.addOverlay(marker);    // 将标注添加到地图中
                    animate && marker.setAnimation(BMAP_ANIMATION_BOUNCE);
                    pointArray.push(point);
                    vm.addClickHandler(curr,marker);
                }   
            }
            //让所有点在视野范围内
            pointArray.length && map.setViewport(pointArray);
        },
        addClickHandler(content,marker){
            let vm = this
            marker.addEventListener("click",function(e){
                vm.openInfo(content,e)
            });
        },
        openInfo(content,e){
            let vm = this
            let sContent =
                "<h4 style='line-height:20px;padding:2px;font-size:14px;'><a style='text-decoration: none;color:#000;display:block;' href='./detail.html?id="+content.id+"'>"+content.name+"</a></h4>" + 
                "<p style='line-height:20px;padding:2px;font-size:12px;'><a style='text-decoration: none;color:#000;display:block;' href='./detail.html?id="+content.id+"'>地址："+content.address+"</a></p>" + 
                "<p style='line-height:20px;padding:2px;font-size:12px;'><a style='text-decoration: none;color:#000;display:block;' href='./detail.html?id="+content.id+"'>备注："+content.description+"</a></p>";
            let p = e.target;
            let spoint = new BMap.Point(p.getPosition().lng, p.getPosition().lat);
            let infoWindow = new BMap.InfoWindow(sContent); 
            vm.map.openInfoWindow(infoWindow,spoint); 
        }
    }
}
</script>
<style lang="scss" scoped>
.bdmap{
    height:100%;
    #bdmap-container{
        position: absolute;
        top: -44px;
        left: 0;
        right: 0;
        bottom: 0;
    }
}
</style>


