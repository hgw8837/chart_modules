<template>
    <div>차트 모듈</div>
    <div>
        <select v-model="theme">
            <option :value="''" disabled>차트 모양</option>
            <option v-for="(item, key) in themeList" :key="key" :value="item">
                {{ item }}
            </option>
        </select>
    </div>
    <div>
        <div v-if="theme == 'HeatMap'">
            <select v-model="selectData">
                <option :value="''" disabled>데이터</option>
                <option v-for="(item, key) in dataList" :key="key" :value="item">
                    {{ item }}
                </option>
            </select>
            <select v-model="columnXY['columnX']">
                <option :value="''" disabled>x축</option>
                <option v-for="(item, key) in categoryDataList" :key="key" :value="item">
                    {{ item }}
                </option>
            </select>
            <select v-model="columnXY['columnY']">
                <option :value="''" disabled>y축</option>
                <option v-for="(item, key) in categoryDataList" :key="key" :value="item">
                    {{ item }}
                </option>
            </select>
            <select v-model="columnXY['columnValue']">
                <option :value="''" disabled>value</option>
                <option v-for="(item, key) in valueDataList" :key="key" :value="item">
                    {{ item }}
                </option>
            </select>
        </div>
        <div v-else>
            <select v-model="columnXY['columnX']">
                <option :value="''" disabled>x축</option>
                <option v-for="(item, key) in categoryDataList" :key="key" :value="item">
                    {{ item }}
                </option>
            </select>
            <select v-model="columnXY['columnY']">
                <option :value="''" disabled>y축</option>
                <option v-for="(item, key) in valueDataList" :key="key" :value="item">
                    {{ item }}
                </option>
            </select>
        </div>
    </div>
    <div class="chart" id="chartdiv" ref="chartdiv" style="width: 100%; height: 700px;"></div>
</template>

<script>
import COMMON_UTIL from "../../../resources/js/commonUtil.js";

import * as am5 from "@amcharts/amcharts5"; // amChart5 기본 라이브러리
import * as am5xy from "@amcharts/amcharts5/xy"; // 막대, 라인 차트
import * as am5percent from "@amcharts/amcharts5/percent"; // 파이류 차트
import am5themes_Animated from "@amcharts/amcharts5/themes/Animated"; // 차트 애니메이션

export default {
    data() {
        return {
            // 차트 정보
            charts: {},

            // 차트 테마 목록
            themeList: ["XY", "YX", "ClusterXY", "ClusterYX", "StackXY", "StackYX", "Line", "ClusterLine", "Pie", "Donut", "HeatMap"],

            // 차트 테마
            theme: "Pie",

            // 사용할 데이터
            dataList: ["data", "heatMapData"],
            selectData: "data",

            // 차트 데이터
            data: [{
                category: "Research",
                category2: "동구",
                value1: 1000,
                value2: 588
            }, {
                category: "Marketing",
                category2: "중구",
                value1: 1200,
                value2: 1800
            }, {
                category: "Sales",
                category2: "남구",
                value1: 850,
                value2: 1230
            }, {
                category: "test",
                category2: "서구",
                value1: 1050,
                value2: 950,
                value3: true
            }],

            heatMapData: [{
                week: "Monday",
                quarter: "0-6시",
                category2: "동구",
                value1: 1000,
                value2: 588
            }, {
                week: "Monday",
                quarter: "7-12시",
                category2: "동구",
                value1: 1200,
                value2: 1800
            }, {
                week: "Monday",
                quarter: "13-18시",
                category2: "동구",
                value1: 850,
                value2: 1230
            }, {
                week: "Monday",
                quarter: "19-24시",
                category2: "동구",
                value1: 1050,
                value2: 950,
            }, {
                week: "Tuesday",
                quarter: "0-6시",
                category2: "중구",
                value1: 2000,
                value2: 458
            }, {
                week: "Tuesday",
                quarter: "7-12시",
                category2: "중구",
                value1: 4000,
                value2: 2100
            }, {
                week: "Tuesday",
                quarter: "13-18시",
                category2: "중구",
                value1: 2500,
                value2: 1430
            }, {
                week: "Tuesday",
                quarter: "19-24시",
                category2: "중구",
                value1: 3100,
                value2: 1150,
            }, {
                week: "Wednesday",
                quarter: "0-6시",
                category2: "동구",
                value1: 1800,
                value2: 1588
            }, {
                week: "Wednesday",
                quarter: "7-12시",
                category2: "중구",
                value1: 4200,
                value2: 900
            }, {
                week: "Wednesday",
                quarter: "13-18시",
                category2: "서구",
                value1: 3500,
                value2: 430
            }, {
                week: "Wednesday",
                quarter: "19-24시",
                category2: "남구",
                value1: 2300,
                value2: 1850,
            }],

            // 데이터 카테고리(x축) 목록 (string)
            categoryDataList: [],

            // 데이터 밸류(y축) 목록 (number)
            valueDataList: [],

            // 차트 초기 축 컬럼
            columnXY: {
                columnX: null,
                columnY: null,
                columnValue: null,
            },

            // 카테고리 목록의 row 데이터 목록 (string)
            categoryRowDataList: {

            },

            // 차트 옵션 변수 목록
            chartOptions: {
                layout: "verticalLayout", // 하위요소(범례 등) 위치 (수평 배치: "horizontalLayout", 수직 배치: "verticalLayout")


                /*************** 글자 (시작) ***************/
                themeFontFamily: "none", // 글꼴 (기본: "none")
                themeFontSize: "1.2rem", // 글자 크기 (기본: "1rem")
                themeFontWeight: "bold", // 글자 두께 (기본: "normal")
                themeFill: "#7b7b7b", // 글자 색상 (기본: "#000000")
                /*************** 글자 (끝) ***************/


                /*************** 그리드 (시작) ***************/
                themeGridVisible: true, // 전체 그리드 표시 여부 (활성화: true, 비활성화: false)
                themeStrokeWidth: 1.5, // 전체 그리드 두께 (기본: 1, 0 ≤ x)
                themeStroke: "#000000", // 전체 그리드 색상 (기본: "#000000")

                minGridDistance: 100, // 밸류축 그리드 밀도 (그리드 간의 최소 픽셀 거리) (기본: undefined, -1 ≤ x ≤ 최댓값)
                /*************** 그리드 (끝) ***************/


                numberUnit: "", // 표기할 숫자 단위 (사용안함: "")

                min: undefined, // 축 최소값, 기본값은 데이터의 최소값 (기본: undefined)
                max: undefined, // 축 최대값, 기본값은 데이터의 최대값 + 10% (기본: undefined)
                strictMinMax: false, // 설정한 min max값 사용 여부, 비활성화 시 설정한 min max값을 차트 모양에 이상적인 값으로 자동 조절 (활성화: true, 비활성화: false)
                extraMin: undefined, // 데이터 최소값에서 (x * 100)% 만큼 축 최소값 범위를 추가, 사용시 축 최소값 비활성화 (기본: undefined, 0.1 ≤ x ≤ 1)
                extraMax: undefined, // 데이터 최대값에서 (x * 100)% 만큼 축 최대값 범위를 추가, 사용시 축 최대값 비활성화  (기본: undefined, 0.1 ≤ x ≤ 1)


                /*************** 마우스 (시작) ***************/
                lineXVisible: true, // 마우스 x축 라인 표시 여부 (활성화: true, 비활성화: false)
                lineYVisible: false, // 마우스 y축 라인 표시 여부 (활성화: true, 비활성화: false)

                panX: true, // 마우스 x축 드래그 이동 여부 (활성화: true, 비활성화: false)
                panY: false, // 마우스 y축 드래그 이동 여부 (활성화: true, 비활성화: false)
                behavior: "none", // 마우스 드래그 확대 범위, 사용 시 드래그 이동 비활성화 (사용안함: "none", x축: "zoomX", y축: "zoomY", xy축: "zoomXY")

                wheelX: "none", // 마우스 휠 수평 회전 시 (사용안함: "none", 가로줌: "zoomX", 세로줌: "zoomY", 가로세로줌: "zoomXY", 가로이동: "panX", 세로이동: "panY", 가로세로이동: "panXY")
                wheelY: "zoomX", // 마우스 휠 수직 회전 시 (사용안함: "none", 가로줌: "zoomX", 세로줌: "zoomY", 가로세로줌: "zoomXY", 가로이동: "panX", 세로이동: "panY", 가로세로이동: "panXY")
                /*************** 마우스 (끝) ***************/


                scrollbarX: true, // x축 스크롤바 표시 여부
                scrollbarY: false, // y축 스크롤바 표시 여부

                categoryAxisTooltip: true, // 카테고리축 툴팁 숨김 여부 (숨김: true, 표시: false)
                valueAxisTooltip: true, // 밸류축 툴팁 숨김 여부 (숨김: true, 표시: false)
                seriesTooltip: false, // 셀 툴팁 숨김 여부 (숨김: true, 표시: false)


                /*************** 셀 (시작) ***************/
                seriesColorList: ["#f9b5a0", "#f0acb2", "#e19dcf", "#d28eed"], // 차트 셀 색상 목록 (기본: [])
                bullet: "Circle", // 글머리기호 (사용안함: "", 원형: "Circle", 텍스트: "Label")

                // 막대 차트
                cornerRadius: 5, // 셀 모서리 둥금 정도 (0 ≤ x)

                // 라인 차트
                lineSeries: "SmoothedXLine", // 라인 차트 종류 (직선: "Line", 곡선: "SmoothedXLine", 계단: "StepLine")
                lineStrokeDasharray: [10, 5], // 점선 설정 (점선 길이, 점선 간격, 두번째 점선 길이, 두번째 점선 간격...) (사용안함: [])
                lineFillHidden: false, // 채우기 비활성화 여부 (활성화: false, 비활성화: true)
                /*************** 셀 (끝) ***************/

                //파이 차트
                donutInnerRadiusPercent: 40, // 도넛 내부 원 크기 (0 ≤ x ≤ 100)
                startAngle: 360, // 차트 시작 각도 (0 ≤ x ≤ 360)
                endAngle: 0, // 차트 끝 각도 (0 ≤ x ≤ 360)
                labelsoprions: "CurcleNear", //라벨 위치(원 밖:"CurcleOut" 원 근처:"CurcleNear" 원 안:"CurcleIn")
                radius: 30, // 차트와 라벨 간의 거리 (기본: 30, ±x)
                textoptions: true, // 라벨 밸류 표시 형식 (%: true, 실제값: false)
                tooltipoptions: false, // 툴팁 밸류 표시 형식 (%: true, 실제값: false)
            }

        };
    },

    methods: {
        // 데이터 전처리 함수
        dataPreProcess: function (data) {
            console.log("dataPreProcess run");

            let value = null; // 객체의 value값 (객체의 크기(개수)를 구하기 위함)
            let type = null; // value의 데이터 타입
            let dataKeys = []; // 데이터 keys (= 컬럼 목록)
            let dataTypes = []; // string, number, etc

            // 데이터 크기만큼 반복
            for (let i in data) {
                value = Object.values(data[i]);

                /* console.log("data[", i, "] length: ", value.length);
                console.log("data ", i, " ", data[i]); */

                // 데이터[i] 객체의 크기(컬럼의 개수)만큼 반복
                for (let j in value) {
                    type = typeof (value[j]);
                    // console.log("value type: ", type);

                    // 데이터 keys 저장 (대부분 데이터[0]에서만 실행)
                    if (value.length > dataTypes.length) {
                        dataKeys[j] = Object.keys(data[i])[j];
                    }

                    // 데이터 타입 분류 (컬럼 데이터에 string이 하나라도 있을경우 해당 컬럼은 string)
                    if (dataTypes[j] == null || dataTypes[j] != "string") {
                        if (type === "string") {
                            dataTypes[j] = "string";
                        } else if (type === "number") {
                            dataTypes[j] = "number";
                        } else {
                            dataTypes[j] = "etc";
                        }
                    }
                }
            }

            /* console.log("dataTypes: ", dataTypes);
            console.log("dataKeys: ", dataKeys); */

            // 축별 컬럼 목록 초기화
            this.categoryDataList = [];
            this.valueDataList = [];
            // 초기 축 컬럼 초기화
            this.columnXY['columnX'] = null;
            this.columnXY['columnY'] = null;
            this.columnXY['columnValue'] = null;
            // 카테고리 row데이터 초기화
            this.categoryRowDataList = {};

            // 데이터 타입이 string이면 x축, number이면 y축 (etc는 보류)
            for (let i in dataTypes) {
                if (dataTypes[i] == "string") {
                    this.categoryDataList.push(dataKeys[i]);
                    // 첫 번째 string 컬럼을 default x축으로 설정
                    if (this.columnXY['columnX'] == null) {
                        this.columnXY['columnX'] = dataKeys[i];
                    }
                    this.categoryRowDataList[dataKeys[i]] = []; // 컬럼명 object의 빈 배열 생성
                } else if (dataTypes[i] == "number") {
                    this.valueDataList.push(dataKeys[i]);
                    // 첫 번째 number 컬럼을 default y축으로 설정
                    if (this.columnXY['columnY'] == null) {
                        this.columnXY['columnY'] = dataKeys[i];
                    }
                }
            }

            /* // x축 목록에 y축도 추가 (x축은 number도 가능)
            this.categoryDataList.push(...this.valueDataList); */

            // 히트맵일 경우 일반 차트와 다르게 x/y/value 값이 필요
            if(this.theme == "HeatMap") {
                this.columnXY['columnValue'] = this.columnXY['columnY'];
                // this.columnXY['columnX'] = this.categoryDataList[0];
                this.columnXY['columnY'] = this.categoryDataList[1];
            }

            /* console.log("categoryDataList: ", this.categoryDataList, this.columnXY['columnX']);
            console.log("valueDataList: ", this.valueDataList, this.columnXY['columnY']);
            console.log("columnValue: ", this.columnXY['columnValue']); */

            // 데이터 타입이 String인 컬럼의 데이터 목록(중복X)
            for(let i in data) {
                // String 컬럼 수만큼 반복
                for(let j in this.categoryDataList) {
                    // console.log("categoryDataList: ", this.categoryDataList[j]);
                    let column = this.categoryRowDataList[this.categoryDataList[j]];
                    // console.log("column: ", column);
                    // row데이터 목록이 비었다면 0번째에 데이터 넣기
                    if(this.categoryRowDataList[this.categoryDataList[j]].length < 1) {
                        column[0] = data[i][this.categoryDataList[j]];
                    }

                    let overlap = 0; // row데이터 중복 검사
                    for(let k in column) {
                        // console.log("row[", i, "] ", data[i][this.categoryDataList[j]]," == column[", k, "]: ", column[k], );
                        if(data[i][this.categoryDataList[j]] == column[k]) {
                            // console.log("overlap: ", column[k]);
                            overlap = 1;
                            break;
                        } else {
                            // console.log("push: ", column[k]);
                        }
                    }
                    
                    // row데이터 목록에 중복이 없다면
                    if(overlap == 0) {
                        column.push(data[i][this.categoryDataList[j]]); // 컬럼에 row 데이터 넣기
                    }
                }
            }
            // console.log("categoryRowDataList: ", this.categoryRowDataList);
        },


        // 차트 생성 실행 함수
        runCreateChart: function (theme, data, columnX, columnY, columnValue, valueDataList, categoryRowDataList, chartOptions) {
            if (theme == "XY" || theme == "YX" || theme == "ClusterXY" || theme == "ClusterYX" || theme == "StackXY" || theme == "StackYX") { // 막대류 차트 생성
                this.createXYChart(theme, data, columnX, columnY, valueDataList, chartOptions);
            } else if (theme == "Line" || theme == "ClusterLine") { // 라인 차트 생성
                this.createLineChart(theme, data, columnX, columnY, valueDataList, chartOptions);
            } else if (theme == "Pie" || theme == "Donut") { // 파이류 차트 생성
                this.createPieChart(theme, data, columnX, columnY, chartOptions);
            } else if (theme == "HeatMap") {
                this.createHeatMapChart(theme, data, columnX, columnY, columnValue, categoryRowDataList, chartOptions);
            } else if (theme == "") {

            } else if (theme == "") {

            }
        },


        // 차트 공통 시작 함수
        commonChartStart: function (options) {
            /******************************************************************************************/
            /*************** 차트 생성 (시작) ***************/
            let chartWarp = this.$refs["chartdiv"]; // 차트 상위 div ref 매칭
            chartWarp.innerHTML = ""; // 차트 상위 div 내용 초기화 (기존 차트 삭제)

            let div = document.createElement("div"); // 차트를 담을 빈 div 생성 (차트 하위 div)
            div.style.width = "100%"; // 차트를 담을 div의 넓이
            div.style.height = "100%"; // 차트를 담을 div의 높이
            chartWarp.appendChild(div); // 차트 상위 div 안에 차트 하위 div를 추가

            let root = am5.Root.new(div); // 차트 하위 div에 차트(root) 담기
            this.charts = root; // 차트 정보 전역에 담기

            // let root = am5.Root.new("chartdiv"); // root 생성 및 id 매칭
            // let root = am5.Root.new(this.$refs.chartdiv); // root 생성 및 ref 매칭
            /*************** 차트 생성 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 차트 전체 옵션 (시작) ***************/
            // root.setThemes([am5themes_Animated.new(root)]); // 툴팁 등 애니메이션 효과

            let myTheme = am5.Theme.new(root);

            myTheme.rule("Label").setAll({ // 전체 글자 설정
                // myTheme.rule("AxisLabel").setAll({ // 축 글자 설정
                fontFamily: options['themeFontFamily'], // 글꼴
                fontSize: options['themeFontSize'], // 글자 크기
                fontWeight: options['themeFontWeight'], // 글자 두께
                fill: am5.color(options['themeFill']), // 글자 색상 (툴팁 제외)
            });

            // 전체 그리드 설정
            myTheme.rule("Grid").setAll({
                visible: options['themeGridVisible'], // 그리드 표시 여부
                strokeWidth: options['themeStrokeWidth'], // 그리드 두께
                stroke: am5.color(options['themeStroke']), // 그리드 색상
            });

            root.setThemes([am5themes_Animated.new(root), myTheme]);
            /*************** 차트 전체 옵션 (끝) ***************/
            /******************************************************************************************/



            // 숫자 형식 설정
            root.numberFormatter.setAll({
                numberFormat: `#,###.##'${options['numberUnit']}`, // 숫자 포맷 형식
                // numericFields: ["valueY"] // 포맷할 필드
            });

            return root;
        },


        // 차트 공통 마지막 함수
        commonChartEnd: function (theme, root, chart, series, options) {
            // 차트 배경
            /* chart.plotContainer.get("background").setAll({
                stroke: am5.color(0x297373), // 외곽선 색상
                strokeOpacity: 0.5, // 외곽선 투명도
                fill: am5.color(0x297373), // 배경 색상
                fillOpacity: 0.2 // 배경 투명도
            }); */



            /******************************************************************************************/
            /*************** 마우스 이벤트 (시작) ***************/
            // 마우스 커서 오버 시 효과
            let cursor = chart.set("cursor", am5xy.XYCursor.new(root, {
                behavior: options['behavior'], // 드래그 확대 활성화 범위 (사용 시 드래그 이동 비활성화)
            }));
            // x축과 연결된 라인 옵션
            cursor.lineX.setAll({
                visible: options['lineXVisible'], // 라인 표시 여부
                // stroke: am5.color(0x550000), // 라인 색상
                // strokeWidth: 2, // 라인 두께
                // strokeDasharray: [20, 5, 5, 5] // 점선 설정 (점선 길이, 점선 간격, 두번째 점선 길이, 두번째 점선 간격...)
            });
            // y축과 연결된 라인 옵션
            cursor.lineY.setAll({
                visible: options['lineYVisible'], // 라인 표시 여부
            });
            /*************** 마우스 이벤트 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 스크롤바 (시작) ***************/
            // x축 스크롤바
            let scrollbarX = chart.set("scrollbarX", am5.Scrollbar.new(root, {
                visible: options['scrollbarX'], // 스크롤바 표시 여부
                orientation: "horizontal", // 스크롤 배치 방향
                minHeight: 8 // 스크롤 너비
            }));
            scrollbarX.startGrip.set("scale", 0.7); // 스크롤 시작 버튼 크기
            scrollbarX.endGrip.set("scale", 0.7); // 스크롤 끝 버튼 크기

            // y축 스크롤바
            let scrollbarY = chart.set("scrollbarY", am5.Scrollbar.new(root, {
                visible: options['scrollbarY'], // 스크롤바 표시 여부
                orientation: "vertical", // 스크롤 배치 방향
                minHeight: 8 // 스크롤 너비
            }));
            scrollbarY.startGrip.set("scale", 0.7); // 스크롤 시작 버튼 크기
            scrollbarY.endGrip.set("scale", 0.7); // 스크롤 끝 버튼 크기
            /*************** 스크롤바 (끝) ***************/
            /******************************************************************************************/


            root._logo.dispose(); //amChart 로고 삭제

            // 히트맵일 시 일반범례 사용 안 하고 return
            if(theme == "HeatMap") {
                return { "root": root, "chart": chart }
            };



            /******************************************************************************************/
            /*************** 범례 (시작) ***************/
            let nameField = null;

            if(theme == "XY") {
                nameField = "categoryX"
            } else if(theme == "YX") {
                nameField = "categoryY"
            }

            let legend = chart.children.push(am5.Legend.new(root, {
                x: am5.percent(50), // 범례 중심 위치 (50: 중앙)
                centerX: am5.percent(50), // 범례 중심 위치 (50: 중앙)
                nameField: nameField, // 카테고리 범례 사용시 필요
                /* layout: am5.GridLayout.new(root, {
                    maxColumns: 3, // 한 줄의 최대 개수
                    fixedWidthGrid: true // 너비가 동일 (활성화: true, 비활성화: false)
                }) */
                /* marginTop: 15,
                marginBottom: 15, */
            }));

            legend.valueLabels.template.set("forceHidden", true); // 범례 밸류값 숨김

            // 범례 아이콘 둥글게
            // legend.markerRectangles.template.setAll({
            //     cornerRadiusTL: 10,
            //     cornerRadiusTR: 10,
            //     cornerRadiusBL: 10,
            //     cornerRadiusBR: 10
            // });

            // 툴팁은 series(전체)에 있고 dataItem(개인)에는 툴팁이 없기 때문에 툴팁 hover가 작동 안 하는 것으로 추정됨
            /* // 범례에 마우스 오버 시 해당 셀 툴팁 표시 (카테고리 범례만)
            legend.itemContainers.template.events.on("pointerover", function (e) {
                let seriesDataItem = e.target.dataItem.dataContext;
                if (seriesDataItem) {
                    let graphics = seriesDataItem.get("graphics");
                    if (graphics) {
                        graphics.hover();
                    }
                }
            });

            // 범례에 마우스 아웃 시 해당 셀 툴팁 미표시 (카테고리 범례만)
            legend.itemContainers.template.events.on("pointerout", function (e) {
                let seriesDataItem = e.target.dataItem.dataContext;
                if (seriesDataItem) {
                    let graphics = seriesDataItem.get("graphics");
                    if (graphics) {
                        graphics.unhover();
                    }
                }
            }); */

            if (theme == "XY" || theme == "YX" || theme == "Pie"|| theme == "Donut") {
                legend.data.setAll(series.dataItems); // 범례 데이터 할당(카테고리)
            } else {
                legend.data.setAll(chart.series.values); // 범례 데이터 할당(value)
            }
            /*************** 범례 (끝) ***************/
            /******************************************************************************************/



            return { "root": root, "chart": chart }
        },


        // 막대류 차트 생성 함수
        createXYChart: function (theme, data, columnX, columnY, valueDataList, chartOptions) {
            let chartData = JSON.parse(JSON.stringify(data)); // 차트 params 데이터 복사
            let options = JSON.parse(JSON.stringify(chartOptions)); // 차트 params 데이터 복사

            // 차트 공통 시작
            let root = this.commonChartStart(options);



            /******************************************************************************************/
            /*************** 차트 테마 (시작) ***************/
            // 마우스 드래그 확대 사용 시 드래그 이동 비활성화
            if (options['behavior'] != "none") {
                options['panX'] = false;
                options['panY'] = false;
            };

            let chart = root.container.children.push(
                am5xy.XYChart.new(root, {
                    panX: options['panX'], // x축 드래그 이동 여부
                    panY: options['panY'], // y축 드래그 이동 여부
                    wheelX: options['wheelX'], // 마우스 휠 수평 회전 시 (사용 안 함)
                    wheelY: options['wheelY'], // 마우스 휠 수직 회전 시
                    layout: root[options['layout']] // 하위요소(범례 등) 위치
                })
            );
            /*************** 차트 테마 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 축 (시작) ***************/
            // 가로 세로 막대에 따라 축 변경
            let categoryAxisRenderer = null;
            let valueAxisRenderer = null;
            let categoryAxes = null;
            let valueAxes = null;
            let categoryInversed = null; // 카테고리 셀 순서 (순차: false, 역순: true)
            if (theme == "XY" || theme == "ClusterXY" || theme == "StackXY") {
                categoryAxisRenderer = "AxisRendererX";
                valueAxisRenderer = "AxisRendererY";
                categoryAxes = "xAxes";
                valueAxes = "yAxes";
                categoryInversed = false;
            } else if (theme == "YX" || theme == "ClusterYX" || theme == "StackYX") {
                categoryAxisRenderer = "AxisRendererY";
                valueAxisRenderer = "AxisRendererX";
                categoryAxes = "yAxes";
                valueAxes = "xAxes";
                categoryInversed = true; // 가로막대차트는 셀이 밑에서부터 시작하므로 위에서부터 보기 위해 역순 적용
            }

            // 축 최대최소값 사용 시 최대최소값 범위 비활성화
            if (options['min'] !== undefined) {
                options['extraMin'] == undefined
            };
            if (options['max'] !== undefined) {
                options['extraMax'] == undefined
            }

            // 카테고리축 렌더링 옵션 (= 가로 막대 y축)
            let categoryRenderer = am5xy[categoryAxisRenderer].new(root, {
                inversed: categoryInversed,
            });
            if (theme == "YX" || theme == "ClusterYX") {
                categoryRenderer.grid.template.setAll({
                    location: 1, // 축 그리드 위치 (0 ≤ x ≤ 1), 가로막대 셀이 역순이라 축 위치도 반대로 옮김
                });
            };

            // 카테고리축 추가
            let categoryAxis = chart[categoryAxes].push(
                am5xy.CategoryAxis.new(root, {
                    categoryField: columnX,
                    renderer: categoryRenderer,
                })
            );
            // 카테고리축 툴팁
            categoryAxis.set("tooltip", am5.Tooltip.new(root, {
                forceHidden: options['categoryAxisTooltip'], // 툴팁 숨김 여부
            }));

            // 밸류축 렌더링 옵션 (= 가로 막대 y축)
            let valueRenderer = am5xy[valueAxisRenderer].new(root, {
                minGridDistance: options['minGridDistance'], // 그리드 간격
            });

            // 밸류축 추가
            let valueAxis = chart[valueAxes].push(
                am5xy.ValueAxis.new(root, {
                    min: options['min'], // 최소값
                    max: options['max'], // 최대값
                    strictMinMax: options['strictMinMax'], // 정확한 최대 최소값 사용 여부
                    extraMin: options['extraMin'], // 최소값 범위
                    extraMax: options['extraMax'], // 최대값 범위
                    renderer: valueRenderer,
                })
            );
            // 밸류축 툴팁
            valueAxis.set("tooltip", am5.Tooltip.new(root, {
                forceHidden: options['valueAxisTooltip'], // 툴팁 숨김 여부
            }));
            /*************** 축 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 그래프 (시작) ***************/
            // 가로 세로 막대에 따라 축 변경
            let xAxis = null;
            let yAxis = null;
            let categoryField = null;
            let valueField = null;
            let categoryXY = null;
            let valueXY = null;
            if (theme == "XY" || theme == "ClusterXY" || theme == "StackXY") {
                xAxis = categoryAxis;
                yAxis = valueAxis;
                categoryField = "categoryXField";
                valueField = "valueYField";
                categoryXY = "categoryX";
                valueXY = "valueY";
            } else if (theme == "YX" || theme == "ClusterYX" || theme == "StackYX") {
                xAxis = valueAxis;
                yAxis = categoryAxis;
                categoryField = "categoryYField";
                valueField = "valueXField";
                categoryXY = "categoryY";
                valueXY = "valueX";
            }

            let series = {};

            // 셀 생성 함수
            function createSeries(name, valueData, seriesColor) {
                // 차트별 툴팁 메세지
                let tooltipText = null;
                if(theme == "XY" || theme == "YX") {
                    tooltipText = `{${categoryXY}}: {${valueXY}}`;
                } else {
                    tooltipText = `${name}: {${valueXY}}`;
                }

                // 셀 생성
                series = chart.series.push(
                    am5xy.ColumnSeries.new(root, {
                        name: name,
                        xAxis: xAxis,
                        yAxis: yAxis,
                        // stacked: stacked,
                        [categoryField]: columnX,
                        [valueField]: valueData,
                        // fill: am5.color("#FFFFFF"), // 셀 베이스 색상(툴팁 사라질 때 배경 색, 베이스 색상(아무색상)을 설정해야 사용자 색상 목록을 세팅할 수 있다)
                        // 셀 툴팁 (동일 카테고리 툴팁 동시 표시)
                        tooltip: am5.Tooltip.new(root, {
                            forceHidden: options['seriesTooltip'], // 툴팁 숨김 여부
                            labelText: tooltipText, // 툴팁 텍스트
                            // pointerOrientation: "vertical", // 툴팁 포인트가 가리키는 방향 ("left", "right", "up", "down", "vertical", "horizontal")
                            // tooltipY: 0, // 셀 위에 위치 (기본: 0, 상승: x < 0, 하락: 0 < x)
                        }),
                        // sequencedInterpolation: true, // 방방 거리는 애니메이션
                    })
                );

                // 다중 차트 툴팁 변경
                if (theme == "ClusterXY" || theme == "ClusterYX" || theme == "StackXY" || theme == "StackYX") {
                    series.get("tooltip").set("labelText", tooltipText);
                };

                // 셀 툴팁 (개별 툴팁 표시(단일 마우스 오버), 툴팁 배경 색 정상, 사용 시 series 안의 셀툴팁/다중차트툴팁 옵션 주석)
                /* series.columns.template.setAll({
                    tooltipText: tooltipText, // 툴팁 텍스트
                    // forceHidden: options['seriesTooltip'], // 툴팁 숨김 맞는지 확인 필요
                    // 툴팁 위치 설정 필요
                }); */

                // 차트에 따른 셀 베이스 색상 (= 툴팁 배경 색상)
                if (theme == "XY" || theme == "YX") {
                    series.set("fill", root.interfaceColors.get("background")); // 차트 배경색과 동일한 색상
                } else {
                    if (seriesColor) {
                        series.set("fill", seriesColor);
                    }
                };

                // 스택 차트 옵션
                if (theme == "StackXY" || theme == "StackYX") {
                    series.set("stacked", true);
                };


                // 셀 모양
                let columnsOption = {};
                if (theme == "XY" || theme == "ClusterXY" || theme == "StackXY") {
                    columnsOption = {
                        cornerRadiusTL: options['cornerRadius'], // 셀 왼쪽 모서리 원형 정도
                        cornerRadiusTR: options['cornerRadius'], // 셀 오른쪽 모서리 원형 정도
                    }
                } else if (theme == "YX" || theme == "ClusterYX" || theme == "StackYX") {
                    columnsOption = {
                        cornerRadiusTR: options['cornerRadius'], // 셀 오른쪽 모서리 원형 정도
                        cornerRadiusBR: options['cornerRadius'], // 셀 왼쪽 모서리 원형 정도
                    }
                };
                series.columns.template.setAll(columnsOption);

                // 셀 색상
                if (theme == "XY" || theme == "YX") {
                    // 차트 default 색상 변경 (색상 목록만큼 순서대로 적용, 색상이 없는 셀은 첫 번째 색상과 비슷한 색상으로 구성)
                    if (seriesColor) {
                        let colorList = [];
                        for (let i = 0; i < options['seriesColorList'].length; i++) {
                            colorList.push(am5.color(options['seriesColorList'][i]));
                        }
                        chart.get("colors").set("colors", colorList);
                    }

                    // 차트 default 색상 목록을 각 셀마다 부여
                    series.columns.template.adapters.add("fill", (fill, target) => {
                        return chart.get("colors").getIndex(series.columns.indexOf(target));
                    });
                    series.columns.template.adapters.add("stroke", (stroke, target) => {
                        return chart.get("colors").getIndex(series.columns.indexOf(target));
                    });


                    // 사용자 색상 목록 (셀 하나씩 색상 지정 가능, series에서 베이스 색상 설정 필요)
                    /* chart.get("colors").set("colors", [
                        am5.color(0x095256),
                        am5.color(0x087f8c),
                        am5.color(0x5aaa95),
                        am5.color(0x86a873),
                        am5.color(0xbb9f06)
                    ]); */
                }

                // 각 막대별 옵션에서 툴팁의 배경색을 바꾸기 (실패)
                /* series.columns.template.adapters.add("tooltip", function(tooltip, target) {
                    console.log("target: ",target.get("tooltip"));
                    if (series.columns.indexOf(target) === 1) {
                        return tooltip.get("background").set("fill", am5.color("#FF0000"));
                    }
                });

                console.log("columns template: ", series.columns.template.entities);
                console.log("columns2 template: ", series.get("tooltip"));
                series.columns.template.get(function(column, target) {
                    console.log("columns template: ", series.columns.indexOf(target));
                }); */

                // 범례에 색상 넣어주기
                if (seriesColor) {
                    series.columns.template.setAll({
                        fill: am5.color(seriesColor),
                        stroke: am5.color(seriesColor),
                    });
                }

                // 글머리기호
                series.bullets.push(function (root) {
                    let lineBulletSprite = {}; // 글머기기호 별 옵션
                    if (options['bullet'] == "Circle") { // 원형 글머리기호
                        lineBulletSprite = {
                            strokeWidth: 2, // 테두리 두께
                            stroke: root.interfaceColors.get("background"), // 테두리 색상
                            radius: 5, // 원형 크기
                            fill: series.get("fill") // 원형 색상
                        }
                    } else if (options['bullet'] == "Label") { // 텍스트 글머리기호
                        lineBulletSprite = {
                            text: `{${columnY}}`, // 텍스트 내용
                            centerX: am5.p50, // 수평 위치, 모든 차트에 적용 (p0, p50, p100)
                            centerY: am5.p50, // 수직 위치, 모든 차트에 적용 (p0, p50, p100)
                            populateText: true // 밸류 필드를 텍스트 내용으로 사용할 때 밸류 값을 사용 (활성화: true, 비활성화: false)
                        }
                    }

                    if (!options['bullet']) { // 글머리기호 미사용 시
                        return null
                    } else {
                        return am5.Bullet.new(root, { // 글머리기호 사용 시
                            locationX: 0.5, // 수평 위치, 차트마다 적용 유무가 다름 (0 ≤ x ≤ 1)
                            locationY: 0.5, // 수직 위치, 차트마다 적용 유무가 다름 (0 ≤ x ≤ 1)
                            sprite: am5[options['bullet']].new(root, lineBulletSprite)
                        });
                    }
                });

                series.data.setAll(chartData); // 차트 내용 데이터 할당

                series.appear(1000); // 차트 내용 등장 시간
            };

            // 셀 생성 함수 실행
            if (theme == "XY" || theme == "YX") {
                createSeries(columnY, columnY, options['seriesColorList'][0]);
            } else if (theme == "ClusterXY" || theme == "ClusterYX" || theme == "StackXY" || theme == "StackYX") {
                // 데이터 밸류 목록만큼 셀 생성
                for (let i = 0; i < valueDataList.length; i++) {
                    createSeries(valueDataList[i], valueDataList[i], options['seriesColorList'][i]);
                }
            }

            /*************** 그래프 (끝) ***************/
            /******************************************************************************************/



            categoryAxis.data.setAll(chartData); // 카테고리축 범주 명 데이터 할당

            // 차트 공통 마지막
            this.commonChartEnd(theme, root, chart, series, options);

            chart.appear(1000, 100); // 차트 배경 등장 시간
        },

        // 라인 차트 생성 함수
        createLineChart: function (theme, data, columnX, columnY, valueDataList, chartOptions) {
            let chartData = JSON.parse(JSON.stringify(data)); // 차트 params 데이터 복사
            let options = JSON.parse(JSON.stringify(chartOptions)); // 차트 params 데이터 복사

            // 차트 공통 시작
            let root = this.commonChartStart(options);



            /******************************************************************************************/
            /*************** 차트 테마 (시작) ***************/
            let chart = root.container.children.push(
                am5xy.XYChart.new(root, {
                    panX: options['panX'], // x축 드래그 이동 여부
                    panY: options['panY'], // y축 드래그 이동 여부
                    wheelX: options['wheelX'], // 마우스 휠 수평 회전 시 (사용 안 함)
                    wheelY: options['wheelY'], // 마우스 휠 수직 회전 시
                    layout: root[options['layout']] // 하위요소(범례 등) 위치
                })
            );
            /*************** 차트 테마 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 축 (시작) ***************/
            // 가로 세로 막대에 따라 축 변경
            let categoryAxisRenderer = null;
            let valueAxisRenderer = null;
            let categoryAxes = null;
            let valueAxes = null;
            let categoryInversed = null; // 카테고리 셀 순서 (순차: false, 역순: true)

            categoryAxisRenderer = "AxisRendererX";
            valueAxisRenderer = "AxisRendererY";
            categoryAxes = "xAxes";
            valueAxes = "yAxes";
            categoryInversed = false;

            // 축 최대최소값 사용 시 최대최소값 범위 비활성화
            if (options['min'] !== undefined) {
                options['extraMin'] == undefined
            };
            if (options['max'] !== undefined) {
                options['extraMax'] == undefined
            }

            // 카테고리축 렌더링 옵션 (= 가로 막대 y축)
            let categoryRenderer = am5xy[categoryAxisRenderer].new(root, {
                inversed: categoryInversed,
            });

            // 카테고리축 추가
            let categoryAxis = chart[categoryAxes].push(
                am5xy.CategoryAxis.new(root, {
                    categoryField: columnX,
                    renderer: categoryRenderer,
                })
            );
            // 카테고리축 툴팁
            categoryAxis.set("tooltip", am5.Tooltip.new(root, {
                forceHidden: options['categoryAxisTooltip'], // 툴팁 숨김 여부
            }));

            // 밸류축 렌더링 옵션 (= 가로 막대 y축)
            let valueRenderer = am5xy[valueAxisRenderer].new(root, {
                minGridDistance: options['minGridDistance'], // 그리드 간격
            });

            // 밸류축 추가
            let valueAxis = chart[valueAxes].push(
                am5xy.ValueAxis.new(root, {
                    min: options['min'], // 최소값
                    max: options['max'], // 최대값
                    strictMinMax: options['strictMinMax'], // 정확한 최대 최소값 사용 여부
                    extraMin: options['extraMin'], // 최소값 범위
                    extraMax: options['extraMax'], // 최대값 범위
                    renderer: valueRenderer,
                })
            );
            // 밸류축 툴팁
            valueAxis.set("tooltip", am5.Tooltip.new(root, {
                forceHidden: options['valueAxisTooltip'], // 툴팁 숨김 여부
            }));
            /*************** 축 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 그래프 생성 (시작) ***************/
            // 가로 세로 막대에 따라 축 변경
            let xAxis = null;
            let yAxis = null;
            let categoryField = null;
            let valueField = null;
            let categoryXY = null;
            let valueXY = null;

            xAxis = categoryAxis;
            yAxis = valueAxis;
            categoryField = "categoryXField";
            valueField = "valueYField";
            categoryXY = "categoryX";
            valueXY = "valueY";

            let series = {};

            function createSeries(name, valueData, seriesColor) {
                series = chart.series.push(
                    am5xy[`${options['lineSeries']}Series`].new(root, {
                        name: name,
                        xAxis: xAxis,
                        yAxis: yAxis,
                        [categoryField]: columnX,
                        [valueField]: valueData,
                        stroke: am5.color(seriesColor), // 선 색상
                        fill: am5.color(seriesColor), // 채우기 색상
                        // minDistance: 10, // 선 단순화 정도 (픽셀 단위만큼의 포인트 생략)
                        // minBulletDistance: 10 // 글머리기호 단순화 정도 (픽셀 단위만큼의 포인트 생략)
                        // maskBullets: true, // 글머리기호 차트 영역 밖에서 잘림 설정 (활성화: true, 비활성화: false)
                        // openValueYField: "value2", // 채우기 영역을 다른 valueField에 맞춤
                        // 셀 툴팁
                        tooltip: am5.Tooltip.new(root, {
                            forceHidden: options['seriesTooltip'], // 툴팁 숨김 여부
                            labelText: `{${categoryXY}}: {${valueXY}}`, // 툴팁 텍스트
                            // pointerOrientation: "vertical", // 툴팁 포인트가 가리키는 방향 ("left", "right", "up", "down", "vertical", "horizontal")
                            // tooltipY: 0, // 셀 위에 위치 (기본: 0, 상승: x < 0, 하락: 0 < x)
                        }),
                        // sequencedInterpolation: true, // 방방 거리는 애니메이션
                    })
                );

                // 다중 차트 툴팁 변경
                if (theme == "ClusterLine") {
                    series.get("tooltip").set("labelText", `${name}: {${valueXY}}`);
                }

                // 선 옵션
                series.strokes.template.setAll({
                    strokeWidth: 2, // 선 굵기
                    strokeDasharray: options['lineStrokeDasharray'], // 점선 설정
                });

                // 채우기 옵션
                series.fills.template.setAll({
                    forceHidden: options['lineFillHidden'], // 채우기 사용 여부
                    visible: true, // 채우기 표시 여부 (시각적으로만 사라지고 옵션은 적용된 상태, 범례 아이콘에 표시됨) (기본: false)
                    fillOpacity: 0.5, // 채우기 투명도
                });

                // 글머리기호
                series.bullets.push(function (root) {
                    let lineBulletSprite = {}; // 글머기기호 별 옵션
                    if (options['bullet'] == "Circle") { // 원형 글머리기호
                        lineBulletSprite = {
                            strokeWidth: 2, // 테두리 두께
                            stroke: root.interfaceColors.get("background"), // 테두리 색상
                            radius: 5, // 원형 크기
                            fill: series.get("fill") // 원형 색상
                        }
                    } else if (options['bullet'] == "Label") { // 텍스트 글머리기호
                        lineBulletSprite = {
                            text: `{${columnY}}`, // 텍스트 내용
                            centerX: am5.percent(50), // 수평 위치, 모든 차트에 적용 (p0, p50, p100)
                            centerY: am5.percent(50), // 수직 위치, 모든 차트에 적용 (p0, p50, p100)
                            populateText: true // 밸류 필드를 텍스트 내용으로 사용할 때 밸류 값을 사용 (활성화: true, 비활성화: false)
                        }
                    }

                    if (!options['bullet']) { // 글머리기호 미사용 시
                        return null
                    } else {
                        return am5.Bullet.new(root, { // 글머리기호 사용 시
                            locationX: 0.5, // 수평 위치, 차트마다 적용 유무가 다름 (0 ≤ x ≤ 1)
                            locationY: 0.5, // 수직 위치, 차트마다 적용 유무가 다름 (0 ≤ x ≤ 1)
                            sprite: am5[options['bullet']].new(root, lineBulletSprite)
                        });
                    }
                });

                series.data.setAll(chartData); // 차트 내용 데이터 할당

                series.appear(1000); // 차트 내용 등장 시간
            }

            // 셀 생성 함수 실행
            if (theme == "Line") {
                createSeries(columnY, columnY, options['seriesColorList'][0]);
            } else if (theme == "ClusterLine") {
                // 데이터 밸류 목록만큼 셀 생성
                for (let i = 0; i < valueDataList.length; i++) {
                    createSeries(valueDataList[i], valueDataList[i], options['seriesColorList'][i]);
                }
            }
            /*************** 그래프 생성 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 그래프 옵션 (시작) ***************/
            /* // 셀 모양
            let columnsOption = {};
            if(theme == "XY") {
                columnsOption = {
                    cornerRadiusTL: options['cornerRadius'], // 셀 왼쪽 모서리 원형 정도
                    cornerRadiusTR: options['cornerRadius'], // 셀 오른쪽 모서리 원형 정도
                }
            } else if(theme == "YX") {
                columnsOption = {
                    cornerRadiusTR: options['cornerRadius'], // 셀 오른쪽 모서리 원형 정도
                    cornerRadiusBR: options['cornerRadius'], // 셀 왼쪽 모서리 원형 정도
                }
            };
            series.columns.template.setAll(columnsOption);

            // 자동 색상 기본 설정
            series.columns.template.adapters.add("fill", function (fill, target) {
                return chart.get("colors").getIndex(series.columns.indexOf(target));
            });
            series.columns.template.adapters.add("stroke", function (stroke, target) {
                return chart.get("colors").getIndex(series.columns.indexOf(target));
            });

            // 셀 색상 (전체 셀을 비슷한 색상으로 구성)
            if(options['seriesColor'] != "none") {
                chart.get("colors").set("colors", [
                    am5.color(options['seriesColor'])
                ]);
            } */

            // 사용자 색상 목록 (셀 하나씩 색상 지정 가능, series에서 베이스 색상 설정 필요)
            /* chart.get("colors").set("colors", [
                am5.color(0x095256),
                am5.color(0x087f8c),
                am5.color(0x5aaa95),
                am5.color(0x86a873),
                am5.color(0xbb9f06)
            ]); */
            /*************** 그래프 옵션 (끝) ***************/
            /******************************************************************************************/



            categoryAxis.data.setAll(chartData); // x축 범주 명 데이터 할당

            // 차트 공통 마지막
            this.commonChartEnd(theme, root, chart, series, options);

            chart.appear(1000, 100); // 차트 배경 등장 시간
        },


        // 파이류 차트 생성 함수
        createPieChart: function (theme, data, columnX, columnY, chartOptions) {
            let chartData = JSON.parse(JSON.stringify(data)); // 차트 params 데이터 복사
            let options = JSON.parse(JSON.stringify(chartOptions)); // 차트 params 데이터 복사

            // 차트 공통 시작
            let root = this.commonChartStart(options);



            /******************************************************************************************/
            /*************** 차트 테마 (시작) ***************/
            let chart = {};
            let donutInnerRadius = false; // 파이 차트는 도넛 사용 안 함

            if (theme == "Donut") {
                donutInnerRadius = am5.percent(options['donutInnerRadiusPercent']);
            }
            chart = root.container.children.push(
                am5percent.PieChart.new(root, {
                    innerRadius: donutInnerRadius, // 도넛 크기
                    layout: root[options['layout']] // 하위요소(범례 등) 위치,
                })
            );
            /*************** 차트 테마 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 그래프 생성 (시작) ***************/
            let series = {};
            let usetext = null;// 라벨텍스트의 값
            let usetooltip = null;// 툴팁텍스트의 값
            let alignLabelss = null;//true 라벨이 false 도형근처 
            let insides = null;//그래프 안에 넣을것이냐 말것이냐
            let ticksoptions = null;// 선의 유무


             //라벨 위치 옵션
             if ((options["labelsoprions"]) == "CurcleOut") {
                alignLabelss = true,
                    insides = false,
                    ticksoptions = false
            }
            else if ((options["labelsoprions"]) == "CurcleNear") {
                alignLabelss = false,
                    insides = false,
                    ticksoptions = true
            }
            else if ((options["labelsoprions"]) == "CurcleIn") {
                alignLabelss = false,
                    insides = true,
                    ticksoptions = true
            }

            series = chart.series.push(
                am5percent.PieSeries.new(root, {
                    name: "파이류 차트",
                    valueField: columnY,
                    categoryField: columnX,
                    startAngle: options["startAngle"], // 차트 시작 각도
                    endAngle: options["endAngle"], // 차트 끝 각도
                    alignLabels: alignLabelss, // 라벨 정렬 상태
                    // fillField: "color"
                    /* legendLabelText: "[{fill}]{category}[/]", // 범례 라벨 텍스트
                    legendValueText: "[bold {fill}]{value}[/]" // 범례 밸류 텍스트 (실제값: {value}, %값: {valuePercentTotal.formatNumber('0.00p')} ... ) */
                })
            );

            // 라벨, 툴팁 밸류 표시 형식
            if (options['textoptions']) {
                usetext = "{category}: {valuePercentTotal.formatNumber('0.00')}%"; // %
            } else {
                usetext = "{category}: {value}" // 실제값
            }
            if(options['tooltipoptions']) {
                usetooltip = "{category}: {valuePercentTotal.formatNumber('0.00')}%" // %
            } else {
                usetooltip = "{category}: {value}" // 실제값
            }

            // 라벨 옵션
            series.labels.template.setAll({
                text: usetext, // 라벨 텍스트
                textType: "circular", //radial 방사형 circular 그래프에 맞게
                inside: insides, //그래프 안에 넣을것이냐 말것이냐.
                radius: options["radius"], //라벨의 거리
            });

            // 라벨의 선
            series.ticks.template.setAll({
                forceHidden: ticksoptions // 라벨의 선의 비활성화 여부
            })

            //셀 옵션
            series.slices.template.setAll({
                tooltipText: usetooltip, //텍스트의 값을 결정 %() or value
                forceHidden: options['lineFillHidden'], // 채우기 비활성화 여부
                /* fillOpacity: 0.5, // 채우기 투명도
                strokeWidth: 2, // 선 굵기 */
            });

            // 차트 기본 색상 목록을 사용자 색상으로 변경
            let colorlist =[];
            for(let i=0;i<options.seriesColorList.length;i++){
                colorlist[i]= am5.color(options.seriesColorList[i]);
            }

            series.get("colors").set("colors", colorlist);

            /*************** 그래프 생성 (끝) ***************/
            /******************************************************************************************/

            series.data.setAll(chartData); // 차트 내용 데이터 할당

            // 차트 공통 마지막
            this.commonChartEnd(theme, root, chart, series, options);

            chart.appear(1000, 100); // 차트 배경 등장 시간
            series.appear(1000); // 차트 내용 등장 시간
        },


        // 히트맵 차트 생성 함수
        createHeatMapChart: function (theme, data, columnX, columnY, columnValue, categoryRowDataList, chartOptions) {
            let chartData = JSON.parse(JSON.stringify(data)); // 차트 params 데이터 복사
            let options = JSON.parse(JSON.stringify(chartOptions)); // 차트 params 데이터 복사

            // 차트 공통 시작
            let root = this.commonChartStart(options);



            /******************************************************************************************/
            /*************** 차트 테마 (시작) ***************/
            // 마우스 드래그 확대 사용 시 드래그 이동 비활성화
            if (options['behavior'] != "none") {
                options['panX'] = false;
                options['panY'] = false;
            };

            let chart = root.container.children.push(
                am5xy.XYChart.new(root, {
                    panX: options['panX'], // x축 드래그 이동 여부
                    panY: options['panY'], // y축 드래그 이동 여부
                    wheelX: options['wheelX'], // 마우스 휠 수평 회전 시 (사용 안 함)
                    wheelY: options['wheelY'], // 마우스 휠 수직 회전 시
                    layout: root[options['layout']] // 하위요소(범례 등) 위치
                })
            );
            /*************** 차트 테마 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 축 (시작) ***************/
            // 가로 세로 막대에 따라 축 변경
            let categoryAxisRenderer = null;
            let valueAxisRenderer = null;
            let categoryAxes = null;
            let valueAxes = null;
            let categoryInversed = null; // 카테고리 셀 순서 (순차: false, 역순: true)
            categoryAxisRenderer = "AxisRendererX";
            valueAxisRenderer = "AxisRendererY";
            categoryAxes = "xAxes";
            valueAxes = "yAxes";
            categoryInversed = false;

            // 카테고리축 렌더링 옵션 (= 가로 막대 y축)
            let categoryRenderer = am5xy[categoryAxisRenderer].new(root, {
                visible: false,
                minGridDistance: 30,
                opposite:true
            });

            // 카테고리축 추가
            let categoryAxis = chart[categoryAxes].push(
                am5xy.CategoryAxis.new(root, {
                    categoryField: columnX,
                    renderer: categoryRenderer,
                })
            );
            // 카테고리축 툴팁
            categoryAxis.set("tooltip", am5.Tooltip.new(root, {
                forceHidden: options['categoryAxisTooltip'], // 툴팁 숨김 여부
            }));

            // 밸류축 렌더링 옵션 (= 가로 막대 y축)
            let valueRenderer = am5xy[valueAxisRenderer].new(root, {
                visible: false,
                minGridDistance: 20,
                inversed: true
            });

            // 밸류축 추가
            let valueAxis = chart[valueAxes].push(
                am5xy.CategoryAxis.new(root, {
                    categoryField: columnY,
                    renderer: valueRenderer,
                })
            );
            // 밸류축 툴팁
            valueAxis.set("tooltip", am5.Tooltip.new(root, {
                forceHidden: options['valueAxisTooltip'], // 툴팁 숨김 여부
            }));
            /*************** 축 (끝) ***************/
            /******************************************************************************************/



            /******************************************************************************************/
            /*************** 그래프 생성 (시작) ***************/
            let xAxis = null;
            let yAxis = null;
            let categoryField = null;
            let categorySecondField = null;
            let valueField = null;
            let categoryXY = null;
            let valueXY = null;
            let calculateAggregates = null;
            let minValue = null;
            let maxValue = null;
            xAxis = categoryAxis;
            yAxis = valueAxis;
            categoryField = "categoryXField";
            categorySecondField = "categoryYField";
            valueField = "valueField";
            categoryXY = "categoryX";
            valueXY = "valueY";
            calculateAggregates = true;

            if(options['min'] !== undefined && options['max'] !== undefined) {
                calculateAggregates = false;
                minValue = options['min'];
                maxValue = options['max'];
            }

            let series = {};

            // 셀 생성 함수
            function createSeries(name, valueData, seriesColor) {
                series = chart.series.push(
                    am5xy.ColumnSeries.new(root, {
                        name: name,
                        xAxis: xAxis,
                        yAxis: yAxis,
                        [categoryField]: columnX,
                        [categorySecondField]: columnY,
                        [valueField]: valueData,
                        calculateAggregates: calculateAggregates, // 집계 계산 (사용: true, 미사용:false, 최소 최대값을 설정할 시 false 처리됨)
                        stroke: am5.color(0xffffff),
                        clustered: false,
                        // fill: am5.color("#FFFFFF"), // 셀 베이스 색상(툴팁 사라질 때 배경 색, 베이스 색상(아무색상)을 설정해야 사용자 색상 목록을 세팅할 수 있다)
                        // 셀 툴팁
                        /* tooltip: am5.Tooltip.new(root, {
                            forceHidden: options['seriesTooltip'], // 툴팁 숨김 여부
                            labelText: `{${categoryXY}}: {${valueXY}}`, // 툴팁 텍스트
                            // pointerOrientation: "vertical", // 툴팁 포인트가 가리키는 방향 ("left", "right", "up", "down", "vertical", "horizontal")
                            // tooltipY: 0, // 셀 위에 위치 (기본: 0, 상승: x < 0, 하락: 0 < x)
                        }), */
                        // sequencedInterpolation: true, // 방방 거리는 애니메이션
                    })
                );

                series.columns.template.setAll({
                    tooltipText: "{categoryX} {categoryY}: {value}", // 툴팁 텍스트
                    strokeOpacity: 1, // 셀 선 불투명도
                    strokeWidth: 2, // 셀 선 두께
                    width: am5.percent(100), // 셀 넓이
                    height: am5.percent(100) // 셀 높이
                });

                // 히트 최소/최대값 색상 세팅
                let startColor = "#fffb77"; // 최소값 기본 색상: 노란색
                let endColor = "#fe131a"; //  최대값 기본 색상: 빨간색
                // 설정 색상 목록에 2개 이상일 시
                if(options['seriesColorList'].length > 1) {
                    startColor = options['seriesColorList'][0]; // 목록의 첫 색상을 최소값 색상
                    endColor = options['seriesColorList'][options['seriesColorList'].length - 1]; // 목록의 끝 색상을 최대값 색상
                }

                // 히트 범례
                let heatLegend = chart.bottomAxesContainer.children.push(am5.HeatLegend.new(root, {
                    orientation: "horizontal", // 범례 방향 (수평: "horizontal", 수직: "vertical")
                    startColor: am5.color(startColor),
                    endColor: am5.color(endColor),
                    // startText: "Lowest", // 최소값 범례 텍스트
                    // endText: "Highest", // 최대값 범례 텍스트
                    // stepCount: 10 // 히트맵 단계
                }));
                
                // 범례 최소/최대값
                series.events.on("datavalidated", function() {
                    if(options['min'] !== undefined && options['max'] !== undefined) {
                        heatLegend.set("startValue", options['min']);
                        heatLegend.set("endValue", options['max']);
                    } else {
                        heatLegend.set("startValue", series.getPrivate("valueLow"));
                        heatLegend.set("endValue", series.getPrivate("valueHigh"));
                    }
                });
                
                // 범례 최소/최대값 색상
                series.set("heatRules", [{
                    target: series.columns.template,
                    min: am5.color(startColor),
                    max: am5.color(endColor),
                    minValue: minValue,
                    maxValue: maxValue,
                    dataField: "value",
                    key: "fill"
                }]);
                
                // 셀 마우스 오버시 범례값 표시
                series.columns.template.events.on("pointerover", function(event) {
                    let di = event.target.dataItem;
                    if (di) {
                        heatLegend.showValue(di.get("value", 0));
                    }
                });

                series.data.setAll(chartData); // 차트 내용 데이터 할당
            }

            createSeries(columnValue, columnValue, options['seriesColorList'][0]);
            /*************** 그래프 생성 (끝) ***************/
            /******************************************************************************************/

            

            /******************************************************************************************/
            /*************** 그래프 옵션 (시작) ***************/

            /*************** 그래프 옵션 (끝) ***************/
            /******************************************************************************************/

            // 축별 범주 명 데이터
            let categoryAxisData = [];
            let valueAxisData = [];
            for(let i in categoryRowDataList[columnX]) {
                categoryAxisData.push({[columnX]: categoryRowDataList[columnX][i]});
            }
            for(let i in categoryRowDataList[columnY]) {
                valueAxisData.push({[columnY]: categoryRowDataList[columnY][i]});
            }

            /* console.log("categoryAxisData: ", categoryAxisData);
            console.log("valueAxisData: ", valueAxisData); */

            // 카테고리축 범주 명 데이터 할당
            categoryAxis.data.setAll(categoryAxisData);

            // 밸류축 범주 명 데이터 할당
            valueAxis.data.setAll(valueAxisData);

            // 차트 공통 마지막
            this.commonChartEnd(theme, root, chart, series, options);

            chart.appear(1000, 100); // 차트 배경 등장 시간
            series.appear(1000); // 차트 내용 등장 시간
        },
    },

    watch: {
        // 차트 테마 변경 시 차트 재생성
        theme: function (newValue, oldValue) {
            // console.log("theme watch: ", newValue, oldValue);
            if (this.selectData != "data") {
                this.selectData = "data";
            } else {
                if(newValue == "HeatMap" || oldValue == "HeatMap") {
                    this.dataPreProcess(this[this.selectData]);
                } else {
                    this.runCreateChart(this.theme, this[this.selectData], this.columnXY['columnX'], this.columnXY['columnY'], this.columnXY['columnValue'], this.valueDataList, this.categoryRowDataList, this.chartOptions);
                }
            }
        },

        // x y축 둘 중 하나라도 변경될 시 차트 재생성
        columnXY: {
            deep: true, // 객체 속 중첩된 값의 변화 감지 (객체 전체 변화 감지)
            handler: function (newValue, oldValue) {
                // console.log("x y watch: ", newValue, oldValue);
                    this.runCreateChart(this.theme, this[this.selectData], this.columnXY['columnX'], this.columnXY['columnY'], this.columnXY['columnValue'], this.valueDataList, this.categoryRowDataList, this.chartOptions);
            },
        },

        // 차트 테마 변경 시 차트 재생성
        selectData: function (newValue, oldValue) {
            console.log("selectData watch: ", newValue, oldValue);
            this.dataPreProcess(this[newValue]);
        },
    },

    computed: {

    },

    mounted() {
        this.dataPreProcess(this.data);
    }
}
</script>

<style scoped></style>